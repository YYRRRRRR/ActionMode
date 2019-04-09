# ActionMode
实验三ActionMode
有参考博客(https://blog.csdn.net/dzc372787439/article/details/44938641
)
MainActivity.java
```
package com.example.hp.actionmode;

import java.util.ArrayList;
import java.util.Iterator;

import android.app.Activity;
import android.graphics.ColorSpace;
import android.os.Bundle;
import android.view.ActionMode;
import android.view.Menu;
import android.view.MenuInflater;
import android.view.MenuItem;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AbsListView;
import android.widget.AbsListView.MultiChoiceModeListener;
import android.widget.BaseAdapter;
import android.widget.CheckBox;
import android.widget.ListView;
import android.widget.TextView;

import com.haarman.listviewanimations.itemmanipulation.AnimateDismissAdapter;
import com.haarman.listviewanimations.itemmanipulation.OnDismissCallback;
import com.example.hp.actionmode.R;

public class MainActivity extends Activity {

    protected static final String TAG="TAG";
    private ListView mListView;
    private MultyAdapter mAdapter;
    // 是否处于ActionMode模式
    private boolean isInActionMode;
    private boolean isInDeleteMode = false;
    private AnimateDismissAdapter<Model> mAnimateDismissAdapter;
    private ArrayList<Integer> mCheckedPositions;

    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //初始化ListView控件
        mListView=(ListView) findViewById(R.id.lv);
        mAdapter=new MultyAdapter();
        mCheckedPositions=new ArrayList<Integer>();

        mAnimateDismissAdapter=new AnimateDismissAdapter<MainActivity.Model>(
                mAdapter,new MyDismissCallBack());
        mAnimateDismissAdapter.setAbsListView(mListView);
        mListView.setAdapter(mAnimateDismissAdapter);

        mListView.setChoiceMode(ListView.CHOICE_MODE_MULTIPLE_MODAL);

        mListView.setMultiChoiceModeListener(new MultiChoiceModeListener() {

            @Override
            public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
                return false;
            }

            @Override
            public void onDestroyActionMode(ActionMode mode) {

                // 在退出ActionMode的时候调用，如果处于删除状态，就删除选中的数据，
                // 否则，重置所有选中的状态
                if (!isInDeleteMode) {
                    for (Model model : mAdapter.models) {
                        model.setChecked(false);
                    }
                    mCheckedPositions.clear();
                }

                isInActionMode = false;

            }

            @Override
            public boolean onCreateActionMode(ActionMode mode, Menu menu) {
                // 在进入ActionMode的时候调用
                MenuInflater inflater = mode.getMenuInflater();
                inflater.inflate(R.menu.meun_delete, menu);
                mode.setTitle("Delete");
                isInActionMode = true;
                isInDeleteMode = false;

                return true;
            }

            @Override
            public boolean onActionItemClicked(ActionMode mode, MenuItem item){

                //当listview中的item被点击的时候调用
                if(item.getItemId()==R.id.action_delete){
                    mAnimateDismissAdapter.animateDismiss(mCheckedPositions);
                    isInDeleteMode = true;
                    mode.finish();
                    return true;
                }

                return false;
            }

            @Override
            public void onItemCheckedStateChanged(ActionMode mode,
                       int position, long id, boolean checked){

                //当item的选中状态被选中的时候调用
                mAdapter.models.get(position).setChecked(checked);
                mAdapter.notifyDataSetChanged();
                mode.setSubtitle(mListView.getCheckedItemCount()
                        +"item selected");

                if(mCheckedPositions.contains(position) && !checked){
                    mCheckedPositions.remove(Integer.valueOf(position));
                }else{
                    mCheckedPositions.add(position);
                }

            }
        });

    }

    private class MultyAdapter extends BaseAdapter{

        private ArrayList<Model>models;

        public MultyAdapter(){
            models=new ArrayList<Model>();
            models.add(new Model("One"));
            models.add(new Model("Two"));
            models.add(new Model("Three"));
            models.add(new Model("Four"));
            models.add(new Model("Five"));
        }

        @Override
        public int getCount() {
            return models.size();
        }

        @Override
        public Model getItem(int position) {
            return models.get(position);
        }

        @Override
        public long getItemId(int position) {
            return 0;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup parent){

            ViewHolder viewHolder;
            Model model=mAdapter.models.get(position);
            if(convertView==null){
                convertView=getLayoutInflater().inflate(R.layout.list_item,parent,false);

                viewHolder = new ViewHolder();

                viewHolder.header=(TextView)convertView.findViewById(R.id.header);
                viewHolder.chb=(CheckBox)convertView.findViewById(R.id.header);
                convertView.setTag(viewHolder);
            }else{
                viewHolder=(ViewHolder)convertView.getTag();
            }
            viewHolder.header.setText(model.getTitle());
            viewHolder.chb.setChecked(model.isChecked());
            viewHolder.chb.setVisibility(isInActionMode ? View.VISIBLE
                    : View.GONE);

            return convertView;
        }
    }

    private static class ViewHolder{
        TextView header;
        CheckBox chb;
    }

    //测试Model
    private class Model{
        private String title;
        private boolean isChecked;
        public Model(String title) {
            this.title = title;
            isChecked = false;
        }
        public String getTitle() {
            return title;
        }

        public boolean isChecked() {
            return isChecked;
        }

        public void setChecked(boolean isChecked) {
            this.isChecked = isChecked;
        }
    }
    private class MyDismissCallBack implements OnDismissCallback {

        @Override
        public void onDismiss(AbsListView arg0, int[] arg1) {

            mCheckedPositions.clear();

            Iterator<Model> iterator = mAdapter.models.iterator();
            while (iterator.hasNext()) {
                if (iterator.next().isChecked()) {
                    // 删除选中的元素
                    iterator.remove();
                }
            }
            mAdapter.notifyDataSetChanged();
        }
    }
}
```
list_item.xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/rl_list_background">

    <ImageView
        android:id="@+id/image"
        android:layout_width="48dp"
        android:layout_height="48dp"
        android:background="@mipmap/ic_launcher"
        android:layout_alignParentLeft = "true"
        />

    <TextView
        android:id="@+id/header"
        android:layout_width="75dp"
        android:layout_height="45dp"
        android:layout_alignParentStart="true"
        android:layout_alignParentTop="true"

        android:layout_marginStart="48dp"
        android:layout_marginTop="0dp"
        android:gravity="center|left"
        android:listSelector="#1C86EE"
        android:textColor="@color/selector"
        android:textSize="16sp" />
    
    <CheckBox
        android:id="@+id/chb"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerVertical="true" />

</RelativeLayout >
menu_delete.xml
```
<menu xmlns:android="http://schemas.android.com/apk/res/android" >

    <item
        android:id="@+id/action_delete"
        android:title="@string/detele"
        android:icon="@android:drawable/ic_menu_delete"
        android:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_edit"
        android:title="@string/detele"
        android:icon="@android:drawable/ic_menu_edit"
        android:showAsAction="always" />
    <item
        android:id="@+id/action_share"
        android:title="@string/detele"
        android:icon="@android:drawable/ic_menu_share"
        android:showAsAction="always" />

</menu>

<!-- android:showAsAction="ifRoom" ifRoom表示,如果有足够的控件，这个值会使菜单显示在ActionBar上-->
<!--android:showAsAction="always" 这个值会使菜单显示在ActionBar上-->
```
结果截图:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190409215228812.png)
最后在真机上运行的时候一直出不来有点问题，后面如果有改出来，进一步更新。
