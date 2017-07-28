最近项目中遇到了这么个需求，妈的竟然和高德地图实现一模一样的功能。因为保密性原则，我就直接上高德地图的截图了。


首先这么一步操作，输入起始点，

之后呢，就进入这个界面





看到这里，大家应该清楚我说的需求吧，好吧，不讲逻辑，单单讲一下这个界面 实现。因为数据都是动态生成的，每次搜索的结果对应的list的大小和内容都是不一样的，所以呢，我们做这个界面的时候，也需要遵循数据源定的游戏规则，我们界面上也要做成动态的。并且是可复用的，这样不论是在操作还是性能上，都会很好。

这个布局主要分两大部分，

  1，头部的水平recycle +指示器

  2，头部以下的分级列表

 

好，分析完大的结构，然后我们再来看一下他们的关系，头部的滑动，会影响到下面列表内容的展示，实施更新对应的数据。

所以我们需要对recycle 的滑动做监听，目的是有两个：1，及时更新指示器 ； 2，及时更新分级列表内容。



下面分享一下我个人这个界面的实现过程。将Recycleview 设置成水平滑动模式，很简单，就这么一句话。

hLinearLayoutManager = new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false);

recyclerView.setLayoutManager(hLinearLayoutManager);

恩，这就实现了水平滑动，但是单单实现水平滑动是不够的，滑动的距离要是一个item 的宽度才行，这样才能像高仿的viewpager。那么很关键的东西出现了。Scroller ，处理recycle滑动分页的工具类。我们自定义的Scorller 继承于recycleview 的onscroollIstener。

最开始是看到这位兄台的博客得到的灵感：一行代码让RecyclerView变身ViewPager

因为他没有实现 指示器的功能，所以我在他的基础上，把指示器添加了进来。下面是我自己的Scroller：

public class PagingScrollHelper {

    RecyclerView mRecyclerView = null;

    private RouteTopAdapter myAdapter = null;

    private MyOnScrollListener mOnScrollListener = new MyOnScrollListener();

    private MyOnFlingListener mOnFlingListener = new MyOnFlingListener();
    private int offsetY = 0;
    private int offsetX = 0;

    int startY = 0;
    int startX = 0;

    private PageIndicatorView mIndicatorView = null;

    private int index = 0;

    private int totalPage = 0;


    enum ORIENTATION {
        HORIZONTAL, VERTICAL, NULL
    }

    ORIENTATION mOrientation = ORIENTATION.HORIZONTAL;

    public void setUpRecycleView(RecyclerView recycleView) {
        if (recycleView == null) {
            throw new IllegalArgumentException("recycleView must be not null");
        }
        mRecyclerView = recycleView;
        //处理滑动
        recycleView.setOnFlingListener(mOnFlingListener);
        //设置滚动监听，记录滚动的状态，和总的偏移量
        recycleView.setOnScrollListener(mOnScrollListener);
        //记录滚动开始的位置
        recycleView.setOnTouchListener(mOnTouchListener);
        //获取滚动的方向
        updateLayoutManger();
    }

    public void updateLayoutManger() {
        RecyclerView.LayoutManager layoutManager = mRecyclerView.getLayoutManager();
        if (layoutManager != null) {
            if (layoutManager.canScrollVertically()) {
                mOrientation = ORIENTATION.VERTICAL;
            } else if (layoutManager.canScrollHorizontally()) {
                mOrientation = ORIENTATION.HORIZONTAL;
            } else {
                mOrientation = ORIENTATION.NULL;
            }
            if (mAnimator != null) {
                mAnimator.cancel();
            }
            startX = 0;
            startY = 0;
            offsetX = 0;
            offsetY = 0;

        }

    }

    ValueAnimator mAnimator = null;

    public class MyOnFlingListener extends RecyclerView.OnFlingListener {

        @Override
        public boolean onFling(int velocityX, int velocityY) {
            if (mOrientation == ORIENTATION.NULL) {
                return false;
            }
            //获取开始滚动时所在页面的index
            int p = getStartPageIndex();

            //记录滚动开始和结束的位置
            int endPoint = 0;
            int startPoint = 0;

            //如果是垂直方向
            if (mOrientation == ORIENTATION.VERTICAL) {
                startPoint = offsetY;

                if (velocityY < 0) {
                        p--;
                } else if (velocityY > 0) {
                        p++;

                }
                //更具不同的速度判断需要滚动的方向
                //注意，此处有一个技巧，就是当速度为0的时候就滚动会开始的页面，即实现页面复位
                endPoint = p * mRecyclerView.getHeight();

            } else {
                startPoint = offsetX;
                if (velocityX < 0) {
                        p--;
                } else if (velocityX > 0) {
                        p++;
                }
                endPoint = p * mRecyclerView.getWidth();

            }
            if (endPoint < 0) {
                endPoint = 0;
            }

            //使用动画处理滚动
            if (mAnimator == null) {
                mAnimator = new ValueAnimator().ofInt(startPoint, endPoint);

                mAnimator.setDuration(300);
                mAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                    @Override
                    public void onAnimationUpdate(ValueAnimator animation) {
                        int nowPoint = (int) animation.getAnimatedValue();

                        if (mOrientation == ORIENTATION.VERTICAL) {
                            int dy = nowPoint - offsetY;
                            //这里通过RecyclerView的scrollBy方法实现滚动。
                            mRecyclerView.scrollBy(0, dy);
                        } else {
                            int dx = nowPoint - offsetX;
                            mRecyclerView.scrollBy(dx, 0);
                        }
                    }
                });
                mAnimator.addListener(new AnimatorListenerAdapter() {
                    @Override
                    public void onAnimationEnd(Animator animation) {
                        //回调监听
                        if (null != mOnPageChangeListener) {
                            mOnPageChangeListener.onPageChange(getPageIndex());
                        }
                    }
                });
            } else {
                mAnimator.cancel();
                mAnimator.setIntValues(startPoint, endPoint);
            }

            mAnimator.start();

            return true;
        }
    }

    public class MyOnScrollListener extends RecyclerView.OnScrollListener {
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            //newState==0表示滚动停止，此时需要处理回滚
            if (newState == 0 && mOrientation != ORIENTATION.NULL) {
                boolean move;
                int vX = 0, vY = 0;
                if (mOrientation == ORIENTATION.VERTICAL) {
                    int absY = Math.abs(offsetY - startY);
                    //如果滑动的距离超过屏幕的一半表示需要滑动到下一页
                    move = absY > recyclerView.getHeight() / 2;
                    vY = 0;

                    if (move) {
                        vY = offsetY - startY < 0 ? -1000 : 1000;
                    }

                } else {
                    int absX = Math.abs(offsetX - startX);
                    move = absX > recyclerView.getWidth() / 2;
                    if (move) {
                        vX = offsetX - startX < 0 ? -1000 : 1000;
                    }

                }

                mOnFlingListener.onFling(vX, vY);

            }

        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            //滚动结束记录滚动的偏移量
            offsetY += dy;
            offsetX += dx;
        }
    }

    private MyOnTouchListener mOnTouchListener = new MyOnTouchListener();


    public class MyOnTouchListener implements View.OnTouchListener {

        @Override
        public boolean onTouch(View v, MotionEvent event) {
            //手指按下的时候记录开始滚动的坐标
            if (event.getAction() == MotionEvent.ACTION_DOWN) {
                startY = offsetY;
                startX = offsetX;
            }
            return false;
        }

    }

    private int getPageIndex() {
        int p = 0;
        if (mOrientation == ORIENTATION.VERTICAL) {
            p = offsetY / mRecyclerView.getHeight();
        } else {
            p = offsetX / mRecyclerView.getWidth();
        }
        mIndicatorView.setSelectedPage(p);

        return p;
    }

    private int getStartPageIndex() {
        int p = 0;
        if (mOrientation == ORIENTATION.VERTICAL) {
            p = startY / mRecyclerView.getHeight();
        } else {
            p = startX / mRecyclerView.getWidth();
        }
        return p;
    }

    onPageChangeListener mOnPageChangeListener;

    public void setOnPageChangeListener(onPageChangeListener listener) {
        mOnPageChangeListener = listener;
    }

    public interface onPageChangeListener {
        void onPageChange(int index);
    }

    public void setIndicator(PageIndicatorView indicatorView) {
        this.mIndicatorView = indicatorView;
    }

    public void setAdapter(RecyclerView.Adapter adapter) {
        this.myAdapter = (RouteTopAdapter) adapter;
        update();
    }



    // 更新页码指示器和相关数据
    private void update() {
        int temp = myAdapter.getItemCount();
        if (temp != totalPage) {
            mIndicatorView.initIndicator(temp);
            if (temp < totalPage && index == totalPage) {
                index = temp;

            }
            mIndicatorView.setSelectedPage(index);
            totalPage = temp;
        }
    }


}

上面有详细的注释，这个类如果你想用的话，可以直接去copy。

这个是Indicator类的代码

public class PageIndicatorView extends LinearLayout {

    private Context mContext = null;
    private int dotSize = 15; // 指示器的大小（dp）
    private int margins = 4; // 指示器间距（dp）
    private List<View> indicatorViews = null; // 存放指示器

    public PageIndicatorView(Context context) {
        this(context, null);
    }

    public PageIndicatorView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public PageIndicatorView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init(context);
    }

    private void init(Context context) {
        this.mContext = context;

        setGravity(Gravity.CENTER);
        setOrientation(HORIZONTAL);

        dotSize = dip2px(context, dotSize);
        margins = dip2px(context, margins);
    }

    // 初始化指示器，默认选中第一页

    public void initIndicator(int count) {

        if (indicatorViews == null) {
            indicatorViews = new ArrayList<>();
        } else {
            indicatorViews.clear();
            removeAllViews();
        }
        View view;
        LayoutParams params = new LayoutParams(dotSize, dotSize);
        params.setMargins(margins, margins, margins, margins);
        for (int i = 0; i < count; i++) {
            view = new View(mContext);
            view.setBackgroundResource(android.R.drawable.presence_invisible);
            addView(view, params);
            indicatorViews.add(view);
        }
        if (indicatorViews.size() > 0) {
            indicatorViews.get(0).setBackgroundResource(android.R.drawable.presence_offline);
        }
    }

    //设置选中页
    public void setSelectedPage(int selected) {
        for (int i = 0; i < indicatorViews.size(); i++) {
            if (i == selected) {
                indicatorViews.get(i).setBackgroundResource(android.R.drawable.presence_offline);
            } else {
                indicatorViews.get(i).setBackgroundResource(android.R.drawable.presence_invisible);
            }
        }
    }

}


这两个关键类已经给出。其他的就是自己来实例化Adapter  ，在XML布局中添加 PagerIndicatorView控件。

绑定adapter 和 scroller。即可。


不明白的可以咨询我。
