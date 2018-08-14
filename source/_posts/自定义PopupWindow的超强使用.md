---
title: 自定义PopupWindow的超强使用
date: 2017-05-08 16:41:05
tags: PopupWindow
---
### 先看效果，在TitleBar上面有一个图片的控件，当点击了图片出现了一个可以扩展的PopupWindow，同时可以通过点击事件去做相对应的操作
![jdfw.gif](http://upload-images.jianshu.io/upload_images/5363507-db9d38a622b9e07b.gif?imageMogr2/auto-orient/strip)
##### 所需要的类的：Item adapter listview 还有一个管理的ExtendPopupWindow
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/5363507-a6e7df4efe56894b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
<!--  more  -->
###1、为了可扩展性的话，我们需要在PopupWindow中加载的是一个可以动态设置的item,其实呢，可以在item中设置过多的属性，以满足实际的项目需要（比如说，可以设置一个红点，提醒用户这里面有更新的东西），这里我只设置了一个title，就是和上面的git图中的一个文字的tiitle，
```
package com.example.popupwindow;

import android.content.Context;

/**
 * @author shiming
 * @time 2017/5/11 18:37
 * @desc  可以扩展的item
 */

public class PopupItem {
    private String title;
    private int tag;

    public int getIcon() {
        return icon;
    }

    public void setIcon(int icon) {
        this.icon = icon;
    }

    private int icon;
    private Context context;
    public Context getContext() {
        return context;
    }

    public void setContext(Context context) {
        this.context = context;
    }

    public int getTag() {
        return tag;
    }

    public void setTag(int tag) {
        this.tag = tag;
    }

    public PopupItem(int tag, String title) {
        this.tag = tag;
        this.title = title;
    }
    public PopupItem(Context context,String title, int tag, int icon) {
        this(tag,title);
        this.icon=icon;
        this.context = context;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}

```
###2、PopupMenuExtendAdapter的创建
1、R.layout.nim_popup_menu_list_item，layout布局，可以通过自定义，这里就是简单的使用的l，使用了ViewHolder 去管理
2、通过对数据的获取PopupItem item = mPopupItemList.get(position);设置layout应该显示的内容
```
package com.example.popupwindow;

import android.content.Context;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.BaseAdapter;
import android.widget.ImageView;
import android.widget.TextView;

import java.util.List;

/**
 * @author shiming
 * @time 2017/5/11 19:08
 * @desc  可以动态的设置很多
 */

public class PopupMenuExtendAdapter extends BaseAdapter {
    private Context mContext;
    private List<PopupItem> mPopupItemList;
    private final LayoutInflater mInflater;

    public PopupMenuExtendAdapter(Context context, List<PopupItem> popupItemList) {
        mContext = context;
        mPopupItemList = popupItemList;
        //通过这个方法去拿到 infalter
        mInflater = (LayoutInflater) context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);
    }

    @Override
    public int getCount() {
        return mPopupItemList.size();
    }

    @Override
    public Object getItem(int position) {
        return mPopupItemList.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder cache;
        if (convertView == null) {
             cache =  new ViewHolder();
            convertView = mInflater.inflate(R.layout.nim_popup_menu_list_item, null);
            cache.icon = (ImageView) convertView.findViewById(R.id.popup_menu_icon);
            cache.title = (TextView) convertView.findViewById(R.id.popup_menu_title);
            convertView.setTag(cache);
        } else {
           cache = (ViewHolder) convertView.getTag();
        }
        PopupItem item = mPopupItemList.get(position);
        cache.title.setText(item.getTitle());
        if (item.getIcon() != 0) {
            cache.icon.setVisibility(View.VISIBLE);
            cache.icon.setImageResource(item.getIcon());
        } else {
            cache.icon.setVisibility(View.GONE);
        }
        // 下面代码实现数据绑定
        return convertView;
    }

    private final class ViewHolder {

        public ImageView icon;

        public TextView title;
    }

}
```
###3、PopupMenuExtendListview的创建，其实也是简单的扩张了listview，对listview的宽度重新的测量，代码如下
```
package com.example.popupwindow;

import android.content.Context;
import android.util.AttributeSet;
import android.view.View;
import android.view.ViewGroup;
import android.widget.ListView;

/**
 * @author shiming
 * @time 2017/5/8 19:56
 * @desc popup  Menu  为了可以扩展 这里继承了 listview
 */

public class PopupMenuExtendListview extends ListView {

    public PopupMenuExtendListview(Context context) {
        this(context, null);
    }

    public PopupMenuExtendListview(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    /**
     * 1.View本身大小多少，这由onMeasure()决定；
     * 2.View在ViewGroup中的位置如何，这由onLayout()决定；
     * 3.绘制View，onDraw()定义了如何绘制这个View。
     */
    /**
     * View本身大小多少，这由onMeasure()决定
     * @param widthMeasureSpec 代模式的32 位
     * @param heightMeasureSpec  代模式的32位
     */
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        int width = measureWidthByChilds() + getPaddingLeft() + getPaddingRight();
        //重新的测量这个 精确的模式下面
        super.onMeasure(MeasureSpec.makeMeasureSpec(width,MeasureSpec.EXACTLY), heightMeasureSpec);
    }

    /**
     * 测量listview中的孩子的宽度
     *
     * @return width
     */
    private int measureWidthByChilds() {
        int width = 0;
        View view = null;
        for (int i = 0; i < getAdapter().getCount(); i++) {
            view = getAdapter().getView(i, view, this);
            if (view != null) {
                //为什么要传入的是0
                // *设置与此视图相关联的布局参数。这些供应
                //*参数，以指定此视图应该如何
                // *设置。有许多子viewgroup.layoutparams，这些
                // *对应的ViewGroup负责不同的子类
                view.setLayoutParams(new ViewGroup.LayoutParams(0, 0));
                // MeasureSpec.UNSPECIFIED是未指定尺寸，这种情况不多，
                // 一般都是父控件是AdapterView，通过measure方法传入的模式。
                view.measure(MeasureSpec.UNSPECIFIED, MeasureSpec.UNSPECIFIED);
                if (view.getMeasuredWidth()>width){
                    width=view.getMeasuredWidth();
                }
            }
        }
        return width;
    }
}
```
在写代码的时候如果我把view.setLayoutParams(new ViewGroup.LayoutParams(0, 0));给去掉了，只去view.measure(MeasureSpec.UNSPECIFIED, MeasureSpec.UNSPECIFIED);我会发现始终测出来的值不对，但是我也没有想通为什么要这么去设置，这个是看别人讲解，也不太明白。记得以前实现过一个动画，先要去mease（0,0）；然后去执行才不会出错，哎哎，先给个todo 
###4、关键的交互类，ExtendPopupWindow详情
1、构造方法传入参数,MenuItemClickListener是内部的接口，专门是做来点击的事件。
```
  //当扩张到一定的地步是不是可以滑动
    private boolean scroll = false;
    public ExtendPopupWindow(Context context, List<PopupItem> items,  MenuItemClickListener listener) {
        this(context,items,false,listener);
    }

    public ExtendPopupWindow(Context context, List<PopupItem> items,  boolean scroll, MenuItemClickListener listener) {
        this.context = context;
        this.items = items;
        this.scroll = scroll;
        this.listener = listener;
        init();
    }
  public interface MenuItemClickListener {
         void onItemClick(PopupItem item);
    }
```
2、init()方法做什么

第一步初始化我们的rootview
```
   if (rootView == null) {
            rootView = LayoutInflater.from(context).inflate(R.layout.nim_popup_menu_layout, null);
            ListView listView = (ListView) rootView.findViewById(R.id.popmenu_listview);
            listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {

                @Override
                public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                    if (listener != null) {
                        popWindow.dismiss();
                        listener.onItemClick(items.get(position));
                    }
                }
            });
            adapter = new PopupMenuExtendAdapter(context, items);
            listView.setAdapter(adapter);
        }
        // focusableInTouchMode.这个属性的意思一如字面所述,就是在进入触摸输入模式后,该控件是否还有获得焦点的能力.
        // 可以简单的理解为,用户一旦开始通过点击屏幕的方式输入,手机就进入了"touch mode".
        // focusableInTouchMode这种属性,多半是设给EditText这种即使在TouchMode下,依然需要获取焦点的控件
        rootView.setFocusableInTouchMode(true);
        //监听返回的按钮，然后 隐藏pop
        rootView.setOnKeyListener(new View.OnKeyListener() {
            @Override
            public boolean onKey(View v, int keyCode, KeyEvent event) {
                if (keyCode == KeyEvent.KEYCODE_MENU && popWindow.isShowing()
                        && event.getAction() == KeyEvent.ACTION_DOWN) {
                    popWindow.dismiss();
                    return true;
                }
                return false;
            }
        });
```
以上的rootview其实是一个layout的高度为15dp，其中嵌套了一个我们封装的popupwindow的listview，在给他设置了一个setOnKeyListener监听，判断了按下rootview，判断三种情况的成立的话pupwindow就必须的dismiss掉，KEYCODE_MENU其实就是早期的菜单键，return true；表示拦截这个操作

第二步做什么：初始化我们popwindow
```
  //初始化我们的popuwindow
        if (popWindow == null) {
            popWindow = new PopupWindow(context);
            popWindow.setContentView(rootView);
            popWindow.setWidth(WindowManager.LayoutParams.WRAP_CONTENT);
            if (scroll) {
                popWindow.setHeight(context.getApplicationContext().getResources().getDisplayMetrics().heightPixels * 2 / 3);
            } else {
                popWindow.setHeight(WindowManager.LayoutParams.WRAP_CONTENT);
            }
            popWindow.setTouchable(true);
            popWindow.setBackgroundDrawable(new BitmapDrawable());
            popWindow.setOnDismissListener(new PopupWindow.OnDismissListener() {
                @Override
                public void onDismiss() {
                }
            });
        }
```
以上需要说明的是： popWindow.setBackgroundDrawable(new BitmapDrawable());我们要给他一个bitmap，要不然会有问题，安卓有很多的地方有这个需求，估计理解为安卓的内部的毛病。
####假想个问题，这既然是一个交互的类，那么我们还需要做什么，好吧，我也咩想到，想到，最后看别人的代码，哈哈…………
#####数据的交互，有adapter的对吧，so,暴露方法告诉adapter的数据变了
```
    public void notifyData() {
        adapter.notifyDataSetChanged();
    }
```
#####pop不可能在初始化页面了，就开始显示了吧，so，砸门的控制show()，判null,是否showing,这里有个关键的东西，我们设置了可以滑动，同时要判断一下我们设备的屏幕的方向。
```
   if (popWindow == null) {
            return;
        }
        if (popWindow.isShowing()) {
            popWindow.dismiss();
        } else {
            if (scroll) {// 当可以滚动时，横竖屏切换时，重新确定高度
                Configuration configuration = context.getResources().getConfiguration();
                int ori = configuration.orientation;
                if (ori == Configuration.ORIENTATION_LANDSCAPE) {
                    popWindow.setHeight(context.getApplicationContext().getResources().getDisplayMetrics().widthPixels* 2 / 3);
                } else {
                    popWindow.setHeight(context.getApplicationContext().getResources().getDisplayMetrics().heightPixels  * 2 / 3);
                }
            }
            popWindow.setFocusable(true);
            popWindow.showAsDropDown(v, -10, 0);
        }
```

####Configuration类专门描述手机设备上的配置信息，这些配置信息既包括用户特定的配置项，也包括系统的动态设备配置。  int ori = configuration.orientation;判断是否是横竖屏，我记得没错的话，横竖屏的话，我们的activity是需要重新的走生命周期，只有onnewintent的这个才不会走，是横的话，为了显示的更加人性化，我们要横的2/3。    popWindow.showAsDropDown(v, -10, 0);显示在哪个控件下面，这个v是我们的控件iew，偏移了-10.y没有动，这些可以动态的设置

  ###5、Activity中的代码如下
```
package com.example.popupwindow;

import android.content.Context;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Toast;

import java.util.ArrayList;
import java.util.List;

public class MainActivity extends AppCompatActivity {

    private ArrayList<PopupItem> menuItemList;
    private ExtendPopupWindow popupMenu;
    private View mImg;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mImg = findViewById(R.id.img);
        mImg.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                initPopuptWindow(MainActivity.this, mImg);
            }
        });
    }

    /**
     * 窗体泄漏造成的原因，是我们的view虽然已经有了这个，但是我们在没有通过点击事件
     * 或者其他的动作，去开启这个popwindow的话，我们的窗体就会泄漏
     * 哈哈
     *
     */
    @Override
    protected void onResume() {
        super.onResume();
//        initPopuptWindow(this, mImg);
    }

    private void initPopuptWindow(Context context, View view) {
        if (popupMenu == null) {
            menuItemList = new ArrayList<>();
            popupMenu = new ExtendPopupWindow(context, menuItemList, listener);
        }
        menuItemList.clear();
        menuItemList.addAll(getMoreMenuItems(context));
        popupMenu.notifyData();
        popupMenu.show(view);
    }

    private static List<PopupItem> getMoreMenuItems(Context context) {
        List<PopupItem> moreMenuItems = new ArrayList<PopupItem>();
        moreMenuItems.add(new PopupItem(context,
                "feiji  0", 1,0));
        moreMenuItems.add(new PopupItem(context,
                "feiji  1", 2,0));
        moreMenuItems.add(new PopupItem(context,
                "feiji  2", 3,0));
        moreMenuItems.add(new PopupItem(context,
                "feiji  3", 4,0));
        return moreMenuItems;
    }

    private ExtendPopupWindow.MenuItemClickListener listener = new ExtendPopupWindow.MenuItemClickListener() {

        @Override
        public void onItemClick(final PopupItem item) {
            switch (item.getTag()) {
                case 1:
                    Toast.makeText(MainActivity.this, "feiji  0",Toast.LENGTH_SHORT).show();
                    break;
                case 2:
                    Toast.makeText(MainActivity.this, "feiji  1",Toast.LENGTH_SHORT).show();
                    break;
                case 3:
                    Toast.makeText(MainActivity.this, "feiji  2",Toast.LENGTH_SHORT).show();
                    break;
                case 4:
                    Toast.makeText(MainActivity.this, "feiji  3",Toast.LENGTH_SHORT).show();
                    break;
            }
        }
    };
}

```
以上只说明一点：popupWindow，不可能在初始化就显示了，只能通过事件来显示，要不然会报错窗体泄漏
#谢谢 ！
###github地址：https://github.com/Shimingli/PopupWindow


