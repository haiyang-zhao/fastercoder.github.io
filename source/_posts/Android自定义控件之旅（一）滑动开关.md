title: Android自定义控件之旅（一）滑动开关
date: 2015-12-28 09:33:42
categories: Android开发
tags: Android自定义控件
---



我们的开关可以单击，可以滑动,效果如下图 

![这里写图片描述](http://img.blog.csdn.net/20150407233434779)

 我们写自定义控件时，一般分为下面几步：
 
   1. 自定义View属性。
   2. 在布局文件中指定自定义属性的取值，并在View的构造方法中获取自定义属性的取值。
   3. 重写onMeasure()方法。有时也可以不用重写。
   4. 重写onDraw()方法。


## 一、自定义View属性##
1、在values下创建attrs.xml资源文件

	<?xml version="1.0" encoding="utf-8"?>
    <resources>
    <attr name="btnBackground" format="reference" />
    <attr name="btnSwitch" format="reference" />
    <attr name="switchState" format="boolean" />
    <declare-styleable name="CustomSwitchButton">
        <attr name="btnBackground" />
        <attr name="btnSwitch" />
        <attr name="switchState" />
    </declare-styleable>
    </resources>

**format**指定了该属性的取值类型，有string,color,demension,integer,enum,reference,float,boolean,fraction,flag;
**declare-styleable** 就是我们要自定义控件的属性,name取值必须是自定义控件的类名
    
<!--more-->
## 二、在布局文件中指定自定义属性的取值，并在View的构造方法中获取自定义属性的取值
### 1、在布局文件中指定自定义属性的取值
在layout下创建main_activity.xml

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:switchBtn="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.zhy.customswitchbutton.CustomSwitchButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        switchBtn:btnBackground="@mipmap/switch_background"
        switchBtn:btnSwitch="@mipmap/switch_button"
        switchBtn:switchState="true" />
    </RelativeLayout>

其中**xmlns:switchBtn="http://schemas.android.com/apk/res-auto"**指定命名空间
命名空间的格式为
Android Stutio中 **xmlns:前缀="http://schemas.android.com/apk/res/res-auto"**
eclipse中 **xmlns:前缀="http://schemas.android.com/apk/res/app包名"**
为自定义属性赋值

    switchBtn:btnBackground="@mipmap/switch_background"
    switchBtn:btnSwitch="@mipmap/switch_button"
    switchBtn:switchState="true" />
### 2、在View的构造方法中获取自定义属性的取值
    public class CustomSwitchButton extends View {
    private Bitmap btnBackground;
    private Bitmap btnSwitch;
    private boolean switchState;

    /**
     * 画笔对象
     */
    private Paint paint;

    /**
     * 滑动的距离
     */
    private float offset;
    /**
     * 是否发生拖动
     */
    private boolean isDrag = false;


    /**
     * 代码中new出来的，执行此构造方法
     * @param context
     */
    public CustomSwitchButton(Context context) {
        this(context, null);
    }
    /**
     * xml中使用时，系统会调用此构造方法
     * @param context
     * @param attrs
     */
    public CustomSwitchButton(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

     /**
     * xml中使用时，系统会调用此构造方法
     * @param context
     * @param attrs
     */
    public CustomSwitchButton(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context, attrs);
    }

    private void initView(Context context, AttributeSet attrs) {

        paint = new Paint();
        paint.setAntiAlias(true);//设置坑锯齿

		 /**
         * 获取各个属性的值 
         */
        TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.CustomSwitchButton);
        btnBackground = BitmapFactory.decodeResource(getResources(),
                ta.getResourceId(R.styleable.CustomSwitchButton_btnBackground, 0));
        btnSwitch = BitmapFactory.decodeResource(getResources(),
                ta.getResourceId(R.styleable.CustomSwitchButton_btnSwitch, 0));
        switchState = ta.getBoolean(R.styleable.CustomSwitchButton_switchState, false);
    }
    }

## 三、重写onMeasure()方法,有时也可以不用重写


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(btnBackground.getWidth(), btnBackground.getHeight());
    }

我们的开关很简单，宽高值就是背景图片的大小

##  四、重写onDraw()方法

    @Override
    protected void onDraw(Canvas canvas) {
        /**
         * 绘制背景
         *Bitmap bitmap 要绘制的图像
         *float left 左边距
         *float top  上边距
         *Paint paint 画笔对象
         */
        canvas.drawBitmap(btnBackground, 0, 0, paint);
        /**
         * 绘制开关
         */
        canvas.drawBitmap(btnSwitch, offset, 0, paint);
    }

其中 **offset** 指的是距离左边的偏移距离，是动态变化的

单击事件，

    @Override
    public void onClick(View v) {
		//拖动时，防止和onTouch冲突
        if (!isDrag) {
            switchState = !switchState;
            changeState();
        }

    }

    private void changeState() {
        offset = switchState ? btnBackground.getWidth() - btnSwitch.getWidth() : 0;
        //重绘制界面
        invalidate();
    }

拖动事件

	/**
     * down 事件时的X坐标
     */
    private float firstX;
    /**
     * up 事件时上次的X坐标
     */
    private float lastX;

    /**
     * 重写onTouchEvent实现滑动效果
     *
     * @param event
     * @return
     */
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        super.onTouchEvent(event);
        float curX = event.getX();
        switch (event.getAction()) {

            case MotionEvent.ACTION_DOWN:
                isDrag = false;
                firstX = lastX = curX;
                break;
            case MotionEvent.ACTION_MOVE:
                //判定是否进行了滑动
                if (Math.abs(lastX - firstX) > 5) {
                    isDrag = true;
                }
                float dis = curX - lastX;
                offset += dis;
                lastX = curX;
                break;
            case MotionEvent.ACTION_UP:
                //未滑完时，判定最终的开关状态
                if (isDrag) {
                    //能滑动的最大距离
                    float maxDis = btnBackground.getWidth() - btnSwitch.getWidth();
                    switchState = offset > maxDis / 2 ? true : false;
                    changeState();
                }
                break;
        }

        refreshView();
        return true;
    }

    /**
     * 刷新界面
     */
    private void refreshView() {
        //判断是否已经超出边界
        float maxDis = btnBackground.getWidth() - btnSwitch.getWidth();
        offset = offset < 0 ? 0 : offset;
        offset = offset > maxDis ? maxDis : offset;
        invalidate();
    }


[**下载源码**](https://github.com/zhy060307/CustomSwitchButton)
   



