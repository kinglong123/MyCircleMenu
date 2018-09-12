#需求背景
1、UI：
![图片.png](https://upload-images.jianshu.io/upload_images/4471198-d848052623bda79a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)
2、效果图：
![GIF.gif](https://upload-images.jianshu.io/upload_images/4471198-d4df866548c86c1f.gif?imageMogr2/auto-orient/strip)
#实现分析
1、首先上面是一个半环形，可先现实一个环形菜单。
2、需实现增加menu接口addByView，和刷新所有menu接口addByAllView。
3、应为是环形，只有下办部分可以显示，课根据环形的角度来进行显示控制。
4、需实现menu点击监听回调，设置选中menu接口。



##实现
环形菜单实现可参考CircleMenu。
![20161029220755275.gif](https://upload-images.jianshu.io/upload_images/4471198-4d1a45fe9e69deda.gif?imageMogr2/auto-orient/strip)
分为：
1.调用方式
2.此控件onMeasure方法;
3.onLayout方法的作用；
4.此控件事件机制dispatchTouchEvent的使用；
5.数学计算—一个缓冲角度。

1、调用方式：
```
        myCircleMenuLayout = (UpCircleMenuLayout) findViewById(R.id.id_mymenulayout);
        myCircleMenuLayout.setMenuItemIconsAndTexts(mItemImgs);//一句设置图片
        myCircleMenuLayout.setOnMenuItemClickListener(new UpCircleMenuLayout.OnMenuItemClickListener() {

            @Override
            public void itemClick(int pos) {
                Toast.makeText(MainActivity.this, mItemTexts[pos],
                        Toast.LENGTH_SHORT).show();
                switch (pos) {
                    case 0:
                        initFragment1();
                        setTitle("安全中心");
                        break;
                    case 1:
                        initFragment2();
                        setTitle("特色服务");
                        break;
                    case 2:
                        initFragment3();
                        setTitle("投资理财");
                        break;
                    case 3:
                        initFragment4();
                        setTitle("转账汇款");
                        break;
                    case 4:
                        initFragment5();
                        setTitle("我的账户");
                        break;
                    case 5:
                        initFragment1();
                        setTitle("安全中心");
                        break;
                    case 6:
                        initFragment2();
                        setTitle("特色服务");
                        break;
                    case 7:
                        initFragment3();
                        setTitle("投资理财");
                        break;
                    case 8:
                        initFragment4();
                        setTitle("转账汇款");
                        break;
                    case 9:
                        initFragment5();
                        setTitle("我的账户");
                        break;
                }
            }

            @Override
            public void itemCenterClick(View view) {
                Toast.makeText(MainActivity.this,
                        "you can do something just like ccb  ",
                        Toast.LENGTH_SHORT).show();
            }
        });

    }
```
（2）此控件onMeasure方法讲解：重点讲解迭代测量

```
/**
     * 设置布局的宽高，并策略menu item宽高
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int resWidth = 0;
        int resHeight = 0;
        double startAngle = mStartAngle;

        double angle = 360 / 10;   //我们传入了10个孩子
        /**
         * 根据传入的参数，分别获取测量模式和测量值
         */
        int width = MeasureSpec.getSize(widthMeasureSpec);
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);

        int height = MeasureSpec.getSize(heightMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);

        /**
         * 如果宽或者高的测量模式非精确值
         */
        if (widthMode != MeasureSpec.EXACTLY
                || heightMode != MeasureSpec.EXACTLY) {
            // 主要设置为背景图的高度

            resWidth = getDefaultWidth();

            resHeight = (int) (resWidth * DEFAULT_BANNER_HEIGTH /
                    DEFAULT_BANNER_WIDTH);

        } else {
            // 如果都设置为精确值，则直接取小值；
            resWidth = resHeight = Math.min(width, height);
        }

        setMeasuredDimension(resWidth, resHeight);

        // 获得直径
        mRadius = Math.max(getMeasuredWidth(), getMeasuredHeight());

        // menu item数量
        final int count = getChildCount();
        // menu item尺寸
        int childSize;

        // menu item测量模式
        int childMode = MeasureSpec.EXACTLY;

        // 迭代测量：根据孩子的数量进行遍历，为每一个孩子测量大小，设置监听回调。
        for (int i = 0; i < count; i++) {
            final View child = getChildAt(i);
            startAngle = startAngle % 360;
            if (startAngle > 269 && startAngle < 271 && isTouchUp) {
                mOnMenuItemClickListener.itemClick(i); //设置监听回调。
                mCurrentPosition = i;  //本次使用mCurrentPosition，只是把他作为一个temp变量，可以有更多的使用，比如动态设置每个孩子相隔的角度
                childSize = DensityUtil.dip2px(getContext(), RADIO_TOP_CHILD_DIMENSION);//设置大小
            } else {
                childSize = DensityUtil.dip2px(getContext(), RADIO_DEFAULT_CHILD_DIMENSION);//设置大小
            }
            if (child.getVisibility() == GONE) {
                continue;
            }
            // 计算menu item的尺寸；以及和设置好的模式，去对item进行测量
            int makeMeasureSpec = -1;

            makeMeasureSpec = MeasureSpec.makeMeasureSpec(childSize,
                    childMode);
            child.measure(makeMeasureSpec, makeMeasureSpec);
            startAngle += angle;
        }
//item容器内边距
        mPadding = DensityUtil.dip2px(getContext(), RADIO_MARGIN_LAYOUT);

    }
```

（3）onLayout方法的讲解

```
/**
     * 设置menu item的位置
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        int layoutRadius = mRadius;
        // Laying out the child views
        final int childCount = getChildCount();

        int left, top;
        // menu item 的尺寸
        int cWidth;

        // 根据menu item的个数，计算角度
        float angleDelay = 360 / 10;
        // 遍历去设置menuitem的位置
        for (int i = 0; i < childCount; i++) {
            final View child = getChildAt(i);
              //根据孩子遍历，设置中间顶部那个的大小以及其他图片大小。
            if (mStartAngle > 269 && mStartAngle < 271 && isTouchUp) {
                cWidth = DensityUtil.dip2px(getContext(), RADIO_TOP_CHILD_DIMENSION);
                child.setSelected(true);
            } else {
                cWidth = DensityUtil.dip2px(getContext(), RADIO_DEFAULT_CHILD_DIMENSION);
                child.setSelected(false);
            }

            if (child.getVisibility() == GONE) {
                continue;
            }
             //大于360就取余归于小于360度
            mStartAngle = mStartAngle % 360;

            float tmp = 0;
            //计算图片布置的中心点的圆半径。就是tmp
            tmp = layoutRadius / 2f - cWidth / 2 - mPadding;
            // tmp cosa 即menu item中心点的横坐标。计算的是item的位置，是计算位置！！！
            left = layoutRadius
                    / 2
                    + (int) Math.round(tmp
                    * Math.cos(Math.toRadians(mStartAngle)) - 1 / 2f
                    * cWidth) + DensityUtil
                    .dip2px(getContext(), 1);
            // tmp sina 即menu item的纵坐标
            top = layoutRadius
                    / 2
                    + (int) Math.round(tmp
                    * Math.sin(Math.toRadians(mStartAngle)) - 1 / 2f * cWidth) + DensityUtil
                    .dip2px(getContext(), 8);
         //接着当然是布置孩子的位置啦，就是根据小圆的来布置的
            child.layout(left, top, left + cWidth, top + cWidth);

            // 叠加尺寸
            mStartAngle += angleDelay;
        }
    }
```

计算小圆的思路
![图片.png](https://upload-images.jianshu.io/upload_images/4471198-d3aa8e06613d8be5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


（4）此控件事件机制dispatchTouchEvent的使用：

```
//dispatchTouchEvent是处理触摸事件分发,事件(多数情况)是从Activity的dispatchTouchEvent开始的。执行super.dispatchTouchEvent(ev)，事件向下分发。
    //onTouchEvent是View中提供的方法，ViewGroup也有这个方法，view中不提供onInterceptTouchEvent。view中默认返回true，表示消费了这个事件。
    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        float x = event.getX();
        float y = event.getY();

        getParent().requestDisallowInterceptTouchEvent(true);
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
            //直接就是获取x，y值了，还有一个DownTime（附送）
                mLastX = x;
                mLastY = y;
                mDownTime = System.currentTimeMillis();
                mTmpAngle = 0;
                break;
            case MotionEvent.ACTION_MOVE:
                isTouchUp = false;   //注意isTouchUp 这个标记量！！！
                /**
                 * 获得开始的角度
                 */
                float start = getAngle(mLastX, mLastY);
                /**
                 * 获得当前的角度
                 */
                float end = getAngle(x, y);
                // 如果是一、四象限，则直接end-start，角度值都是正值
                if (getQuadrant(x, y) == 1 || getQuadrant(x, y) == 4) {
                    mStartAngle += end - start;
                    mTmpAngle += end - start;//按下到抬起时旋转的角度
                } else
                // 二、三象限，色角度值是负值
                {
                    mStartAngle += start - end;
                    mTmpAngle += start - end;
                }
                // 重新布局
                if (mTmpAngle != 0) {
                    requestLayout();
                }

                mLastX = x;
                mLastY = y;

                break;
            case MotionEvent.ACTION_UP:
            //当手指UP啦，就是关键啦，一个缓冲角度，即我们将要固定几个位置，而不是任意位置。我们要设计一个可能的角度去自动帮他选择。
                backOrPre();
                break;
        }
        return super.dispatchTouchEvent(event);
    }
```
MotionEvent事件机制：（此控件我只用了三个）主要的事件类型有:ACTION_DOWN: 表示用户开始触摸。ACTION_MOVE: 表示用户在移动(手指或者其他)。ACTION_UP:表示用户抬起了手指。
（5）数学计算—一个缓冲角度。
```
private void backOrPre() {     //缓冲的角度。即我们将要固定几个位置，而不是任意位置。我们要设计一个可能的角度去自动帮他选择。
        isTouchUp = true;
        float angleDelay = 360 / 10;              //这个是每个图形相隔的角度
        //我们本来的上半圆的图片角度应该是：18,54，90,126,162。所以我们这里是：先让当前角度把初始的18度减去再取余每个图形相隔角度。得到的是什么呢？就是一个图片本来应该在的那堆角度。所以如果是就直接return了。
        if ((mStartAngle-18)%angleDelay==0){
            return;
        }
        float angle = (float)((mStartAngle-18)%36);                 //angle就是那个不是18度开始布局，然后是36度的整数的多出来的部分角度
        //以下就是我们做的缓冲角度处理啦，如果多出来的部分角度大于图片相隔角度的一半就往前进一个，如果小于则往后退一个。
        if (angleDelay/2 > angle){
            mStartAngle -= angle;
        }else if (angleDelay/2<angle){
            mStartAngle = mStartAngle - angle + angleDelay;         //mStartAngle就是当前角度啦，取余36度就是多出来的角度，拿这个多出来的角度去数据处理。
        }
        //然后重新布局onlayout
        requestLayout();
    }
```
##半环形实现
上面我们显示了环形菜单
现在我们来实现半环形
1、onMeasure高度的控制中，将宽度设置为
```
 resHeight = (int) (resWidth/2);
```
2、在onLayout 布局控控制中，整体子view UI需要向移动height/2

```
      child.layout(left, top - mRadius / 2, left + cWidth,top + cWidth - mRadius / 2);
```
因为是办环形所以超过0-180°范围的view应该将其隐藏

···
            if (tampStartAngle >= 0 && tampStartAngle <= 180) {
                child.setVisibility(VISIBLE);
            } else {
                child.setVisibility(INVISIBLE);
            }
···


3、在dispatchTouchEvent中，整体子view UI需要向移动height/2，则对滑动的判断需要作出调整

```
    /**
     * 根据当前位置计算象限
     */
    private int getQuadrant(float x, float y) {
        int tmpX = (int) (x - mRadius / 2);
        int tmpY = (int) (y - mRadius / 2);//新增加了 - mRadius / 2
        if (tmpX >= 0) {
            return tmpY >= 0 ? 4 : 1;
        } else {
            return tmpY >= 0 ? 3 : 2;
        }

    }
```

4、数学计算—一个缓冲角度中backOrPre()

```
    private void backOrPre() {     //缓冲的角度。即我们将要固定几个位置，而不是任意位置。我们要设计一个可能的角度去自动帮他选择。
        isTouchUp = true;
        if(mTmpAngle ==0){
            return;
        }
        //因为中间的子view的角度是90度，当停止时需要找出那个子view距离90度最近，再将其设置到中间
        double temp =mStartAngle;//手势放开时的角度
        double tempStart=mStartAngle;
        boolean f = true;
        for(int i=-(int)mAngleInterval*(getChildCount()-1)+90;i<=90;i+=mAngleInterval){
            double temp1 =  Math.abs(mStartAngle-i);
            if(f){
                temp =  temp1;
                f = false;
            }
            if(temp1<=temp){
                tempStart  = i;
                temp = temp1;
            }

        }
        mStartAngle = tempStart;



        requestLayout();
    }
```
这样基本已经实现了半环形设计

###需实现增加menu接口addByView，和刷新所有menu接口addByAllView。
addByView
```
    /**
     *
     *
     * @param view
     */
    public void addByView(View view) {

        mMenuItemCount+=1;//个数相应增加
        addView(view);


    }
```
addByAllView

```
    /**
     *
     *
     * @param views
     */
    public void addByAllView(List<View> views) {

        removeAllViews();//情况view

        mMenuItemCount=views.size();
        mStartAngle = -(int)mAngleInterval*(mMenuItemCount-1) +90; //角度计算

        for (View view:views){
            addView(view);
        }



    }
```

mAngleInterval 是每个子view的角度间隔,可自行设置

```
    public void setAngleInterval(double angleInterval) {
        mAngleInterval = angleInterval;
    }

```


###需实现menu滚动到中间回调，已经设置选中第几个menu接口。

menu滚动到中间回调，只需判断是否是90度。
```
            if (startAngle == 90 && isTouchUp) {
                if (mCurrentPosition == i) {
                } else {
                    if (mOnMenuItemClickListener != null) {
                        mOnMenuItemClickListener.itemClick(count - i - 1);              //设置监听回调。
                    }
                }
                mCurrentPosition= i;      //本次使用mCurrentPosition，只是把他作为一个temp变量。可以有更多的使用，比如动态设置每个孩子相隔的角度
            }
```

设置选中第几个menu接口,需要位置换算角度，然后重新绘制页面
```
    private void setStartAngle(int i) {

        double startAngleTemp = -(int) mAngleInterval * (i) + 90;

        if (mStartAngle == startAngleTemp) {
            return;
        }
        mStartAngle = startAngleTemp;

        requestLayout();
    }
```
这样总统逻辑也就完成了。
###遇到问题
addByAllView时，如果在操作UI时，会出现已经存在的子view无法清除。
原因：
由于之前添加的childview执行了Animation动画，因为帧动画是对childview的重绘，所以，虽然执行过removeAllViews(); 但是帧动画对view的区域并没有清除掉，以至于感觉removeAllViews方法‘失效’，旧的childview还在界面上

解决在使用时，新增
```
         mIdMenu.removeAllViewsInLayout();
```
完成了。


