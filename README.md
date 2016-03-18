## 【译】android中使用RecyclerView

此篇博客为译文，转载请注明出处。

* 原文：[Android Working with Recycler View](http://www.androidhive.info/2016/01/android-working-with-recycler-view/)
* 原文作者：Ravi Tamada
* 译者：leibnik    
* 时间：2016-02-21

[RecyclerView](http://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)
是一个更高级版本的ListView，较后者有着性能的提升和其他的优点。同时使用RecyclerView和[CardView](http://developer.android.com/reference/android/support/v7/widget/CardView.html)来生成列表和表格都是非常简单的事情。[这里](http://developer.android.com/training/material/lists-cards.html)
有关于RecyclerView和一些例子的完整介绍。

在本次教程里，我们将学习如何结合自定义布局渲染出一个简单的RecyclerView。同时，我们将学习如何写出一个适配器类，以及为列表添加分割线，为列表项添加点击事件。
具体的，我们将把这个RecyclerView设计成呈现电影信息的列表，包括标题，类型，以及发行时间。

以下就是包含必要配置的RecyclerView。

```html
<android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:scrollbars="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```

在开始之前，你必须确保将你的Android Studio更新到[最新版本](http://tools.android.com/)。当前我的Android Studio已经更新到了**Android Studio 2.0 Preview 6**(译者注：真够快的)。

###一、新建项目
1. 在android studio中，点击File->New Project，填好所有必填项后创建项目。
当提示选择默认的activity时，选择**Blank Activity**然后继续。

2. 打开**build.gradle**添加RecyclerView的依赖。`com.android.support:recyclerview-v7:23.1.1`，然后rebuild项目。
```java
BUILD.GRADLE
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.1.1'
    compile 'com.android.support:design:23.1.1'
    compile 'com.android.support:recyclerview-v7:23.1.1'
}
```

3. 在最新版本的build tools中，android studio会为每个activity创建两个布局文件。对于MainActivity，它创建了**activity_main.xml**(包含CoordinatorLayout和AppBarLayout)
以及**content_main.xml**(对应实际的内容)。打开**content_main.xml**添加RecyclerView。
```html
CONTENT_MAIN.XML

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:layout_behavior="@string/appbar_scrolling_view_behavior"
    tools:showIn="@layout/activity_main"
    tools:context=".MainActivity">
 
    <android.support.v7.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:scrollbars="vertical" />
 
</RelativeLayout>
```

4. 在res->values路径下打开**colors.xml**并添加以下几种颜色。
```html
COLORS.XML

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorPrimary">#3F51B5</color>
    <color name="colorPrimaryDark">#303F9F</color>
    <color name="colorAccent">#FF4081</color>
    <color name="year">#999999</color>
    <color name="title">#222222</color>
</resources>
```

###二、写适配器类
添加完RecyclerView之后，我们开始写适配器类来填充数据。RecyclerView的适配器和ListView的一样只是重写的方法不同。

5. 新建一个类**Movie**并声明title，genre以及year。同时为每个变量生成相应的getter/setter方法。
```java
MOVIE.JAVA

package info.androidhive.recyclerview;
 
public class Movie {
    private String title, genre, year;
 
    public Movie() {
    }
 
    public Movie(String title, String genre, String year) {
        this.title = title;
        this.genre = genre;
        this.year = year;
    }
 
    public String getTitle() {
        return title;
    }
 
    public void setTitle(String name) {
        this.title = name;
    }
 
    public String getYear() {
        return year;
    }
 
    public void setYear(String year) {
        this.year = year;
    }
 
    public String getGenre() {
        return genre;
    }
 
    public void setGenre(String genre) {
        this.genre = genre;
    }
}
```

6. 使用下面的代码新建一个布局文件**movie_list_row.xml**，这个布局文件通过展示电影名称，类型，发行年份来渲染了RecyclerView中单个的项。
```html
MOVIE_LIST_ROW.XML

<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:focusable="true"
    android:paddingLeft="16dp"
    android:paddingRight="16dp"
    android:paddingTop="10dp"
    android:paddingBottom="10dp"
    android:clickable="true"
    android:background="?android:attr/selectableItemBackground"
    android:orientation="vertical">
 
    <TextView
        android:id="@+id/title"
        android:textColor="@color/title"
        android:textSize="16dp"
        android:textStyle="bold"
        android:layout_alignParentTop="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
    <TextView
        android:id="@+id/genre"
        android:layout_below="@id/title"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
 
    <TextView
        android:id="@+id/year"
        android:textColor="@color/year"
        android:layout_width="wrap_content"
        android:layout_alignParentRight="true"
        android:layout_height="wrap_content" />
 
</RelativeLayout>
```

7. 现在新建一个类**MoviesAdapter.java**并添加如下的代码。其中**onCreateViewHolder()**方法使**movie_list_row.xml**布局得到填充。
在**onBindViewHolder()**方法中将电影信息（标题，类型，年份）设置到每一项中。
```java
MOVIESADAPTER.JAVA

package info.androidhive.recyclerview;
 
import android.support.v7.widget.RecyclerView;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
 
import java.util.List;
 
public class MoviesAdapter extends RecyclerView.Adapter<MoviesAdapter.MyViewHolder> {
 
    private List<Movie> moviesList;
 
    public class MyViewHolder extends RecyclerView.ViewHolder {
        public TextView title, year, genre;
 
        public MyViewHolder(View view) {
            super(view);
            title = (TextView) view.findViewById(R.id.title);
            genre = (TextView) view.findViewById(R.id.genre);
            year = (TextView) view.findViewById(R.id.year);
        }
    }
 
 
    public MoviesAdapter(List<Movie> moviesList) {
        this.moviesList = moviesList;
    }
 
    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View itemView = LayoutInflater.from(parent.getContext())
                .inflate(R.layout.movie_list_row, parent, false);
 
        return new MyViewHolder(itemView);
    }
 
    @Override
    public void onBindViewHolder(MyViewHolder holder, int position) {
        Movie movie = moviesList.get(position);
        holder.title.setText(movie.getTitle());
        holder.genre.setText(movie.getGenre());
        holder.year.setText(movie.getYear());
    }
 
    @Override
    public int getItemCount() {
        return moviesList.size();
    }
}
```

8. 现在打开**MainActivity.java**并依据下面代码做修改。其中**prepareMovieData()**方法为列表添加了样例数据。
```java
MAINACTIVITY.JAVA

package info.androidhive.recyclerview;
 
import android.content.Context;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.view.GestureDetector;
import android.view.MotionEvent;
import android.view.View;
import android.widget.Toast;
 
import java.util.ArrayList;
import java.util.List;
 
public class MainActivity extends AppCompatActivity {
    private List<Movie> movieList = new ArrayList<>();
    private RecyclerView recyclerView;
    private MoviesAdapter mAdapter;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
 
        recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
 
        mAdapter = new MoviesAdapter(movieList);
        RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(getApplicationContext());
        recyclerView.setLayoutManager(mLayoutManager);
        recyclerView.setItemAnimator(new DefaultItemAnimator());
        recyclerView.setAdapter(mAdapter);
 
        prepareMovieData();
    }
 
    private void prepareMovieData() {
        Movie movie = new Movie("Mad Max: Fury Road", "Action & Adventure", "2015");
        movieList.add(movie);
 
        movie = new Movie("Inside Out", "Animation, Kids & Family", "2015");
        movieList.add(movie);
 
        movie = new Movie("Star Wars: Episode VII - The Force Awakens", "Action", "2015");
        movieList.add(movie);
 
        movie = new Movie("Shaun the Sheep", "Animation", "2015");
        movieList.add(movie);
 
        movie = new Movie("The Martian", "Science Fiction & Fantasy", "2015");
        movieList.add(movie);
 
        movie = new Movie("Mission: Impossible Rogue Nation", "Action", "2015");
        movieList.add(movie);
 
        movie = new Movie("Up", "Animation", "2009");
        movieList.add(movie);
 
        movie = new Movie("Star Trek", "Science Fiction", "2009");
        movieList.add(movie);
 
        movie = new Movie("The LEGO Movie", "Animation", "2014");
        movieList.add(movie);
 
        movie = new Movie("Iron Man", "Action & Adventure", "2008");
        movieList.add(movie);
 
        movie = new Movie("Aliens", "Science Fiction", "1986");
        movieList.add(movie);
 
        movie = new Movie("Chicken Run", "Animation", "2000");
        movieList.add(movie);
 
        movie = new Movie("Back to the Future", "Science Fiction", "1985");
        movieList.add(movie);
 
        movie = new Movie("Raiders of the Lost Ark", "Action & Adventure", "1981");
        movieList.add(movie);
 
        movie = new Movie("Goldfinger", "Action & Adventure", "1965");
        movieList.add(movie);
 
        movie = new Movie("Guardians of the Galaxy", "Science Fiction & Fantasy", "2014");
        movieList.add(movie);
 
        mAdapter.notifyDataSetChanged();
    }
}
```

现在如果你运行你的app，你将看到电影的数据以列表的方式展示了。

![](http://ww2.sinaimg.cn/mw690/006nezkigw1f16n7lfcmsj30k00qh0us.jpg)

###三、添加RecyclerView的分割线
RecyclerView并没有任何与分割线直接相关的属性去显示分割线。然而你需要的是继承类[ItemDecoration]()
同时使用**addItemDecoration()**方法来显示分割线。

9. 使用以下代码创建一个类**DividerItemDecoration.java**
```java

DIVIDERITEMDECORATION.JAVA

package info.androidhive.recyclerview;
 
import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Rect;
import android.graphics.drawable.Drawable;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.view.View;
 
/**
 * Created by Lincoln on 30/10/15.
 */
public class DividerItemDecoration extends RecyclerView.ItemDecoration {
 
    private static final int[] ATTRS = new int[]{
            android.R.attr.listDivider
    };
 
    public static final int HORIZONTAL_LIST = LinearLayoutManager.HORIZONTAL;
 
    public static final int VERTICAL_LIST = LinearLayoutManager.VERTICAL;
 
    private Drawable mDivider;
 
    private int mOrientation;
 
    public DividerItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        a.recycle();
        setOrientation(orientation);
    }
 
    public void setOrientation(int orientation) {
        if (orientation != HORIZONTAL_LIST && orientation != VERTICAL_LIST) {
            throw new IllegalArgumentException("invalid orientation");
        }
        mOrientation = orientation;
    }
 
    @Override
    public void onDrawOver(Canvas c, RecyclerView parent, RecyclerView.State state) {
        if (mOrientation == VERTICAL_LIST) {
            drawVertical(c, parent);
        } else {
            drawHorizontal(c, parent);
        }
    }
 
    public void drawVertical(Canvas c, RecyclerView parent) {
        final int left = parent.getPaddingLeft();
        final int right = parent.getWidth() - parent.getPaddingRight();
 
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int top = child.getBottom() + params.bottomMargin;
            final int bottom = top + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }
 
    public void drawHorizontal(Canvas c, RecyclerView parent) {
        final int top = parent.getPaddingTop();
        final int bottom = parent.getHeight() - parent.getPaddingBottom();
 
        final int childCount = parent.getChildCount();
        for (int i = 0; i < childCount; i++) {
            final View child = parent.getChildAt(i);
            final RecyclerView.LayoutParams params = (RecyclerView.LayoutParams) child
                    .getLayoutParams();
            final int left = child.getRight() + params.rightMargin;
            final int right = left + mDivider.getIntrinsicHeight();
            mDivider.setBounds(left, top, right, bottom);
            mDivider.draw(c);
        }
    }
 
    @Override
    public void getItemOffsets(Rect outRect, View view, RecyclerView parent, RecyclerView.State state) {
        if (mOrientation == VERTICAL_LIST) {
            outRect.set(0, 0, 0, mDivider.getIntrinsicHeight());
        } else {
            outRect.set(0, 0, mDivider.getIntrinsicWidth(), 0);
        }
    }
}
```

10. 打开**MainActivity.java**，在为RecycleView设置适配器之前使用`addItemDecoration()`方法设置分割线。
```java
recyclerView.addItemDecoration(new DividerItemDecoration(this, LinearLayoutManager.VERTICAL));
 
// set the adapter
recyclerView.setAdapter(mAdapter);
```

如果现在运行你的app，你将看到分割线成功分割了列表的每一行。

![](http://ww4.sinaimg.cn/mw690/006nezkigw1f16n5zw484j30k00fptad.jpg)

###四、为RecyclerView添加点击事件

RecycleView没有OnItemClickListener方法去识别列表项的点击事件。而你需要做的就是写一个类继承[RecyclerView.OnItemTouchListener](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.OnItemTouchListener.html)。

11. 打开**MainActivity.java**并添加**RecyclerTouchListener**类以及**ClickListener**接口
```java
MAINACTIVITY.JAVA

public interface ClickListener {
        void onClick(View view, int position);
 
        void onLongClick(View view, int position);
}
 
public static class RecyclerTouchListener implements RecyclerView.OnItemTouchListener {

    private GestureDetector gestureDetector;
    private MainActivity.ClickListener clickListener;

    public RecyclerTouchListener(Context context, final RecyclerView recyclerView, final MainActivity.ClickListener clickListener) {
        this.clickListener = clickListener;
        gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
            @Override
            public boolean onSingleTapUp(MotionEvent e) {
                return true;
            }

            @Override
            public void onLongPress(MotionEvent e) {
                View child = recyclerView.findChildViewUnder(e.getX(), e.getY());
                if (child != null && clickListener != null) {
                    clickListener.onLongClick(child, recyclerView.getChildPosition(child));
                }
            }
        });
    }

    @Override
    public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {

        View child = rv.findChildViewUnder(e.getX(), e.getY());
        if (child != null && clickListener != null && gestureDetector.onTouchEvent(e)) {
            clickListener.onClick(child, rv.getChildPosition(child));
        }
        return false;
    }

    @Override
    public void onTouchEvent(RecyclerView rv, MotionEvent e) {
    }

    @Override
    public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {

    }
}
```

最后以如下的代码为RecyclerView添加点击事件。

```java
recyclerView.addOnItemTouchListener(new RecyclerTouchListener(getApplicationContext(), recyclerView, new ClickListener() {
            @Override
            public void onClick(View view, int position) {
                Movie movie = movieList.get(position);
                Toast.makeText(getApplicationContext(), movie.getTitle() + " is selected!", Toast.LENGTH_SHORT).show();
            }
 
            @Override
            public void onLongClick(View view, int position) {
 
            }
        }));
```

运行app然后检验点击事件。当点击其中一行时你应该能看到一条Toast信息弹出来。如果你在lollipop版本的设备上测试时，还能看到波纹效果。

![](http://ww2.sinaimg.cn/mw690/006nezkigw1f16n5ytcnuj30k00hjwfs.jpg)

###最后的代码
以下就是完整的**MainActivity.java**
```java
MAINACTIVITY.JAVA

package info.androidhive.recyclerview;
 
import android.content.Context;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.view.GestureDetector;
import android.view.MotionEvent;
import android.view.View;
import android.widget.Toast;
 
import java.util.ArrayList;
import java.util.List;
 
public class MainActivity extends AppCompatActivity {
    private List<Movie> movieList = new ArrayList<>();
    private RecyclerView recyclerView;
    private MoviesAdapter mAdapter;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
 
        recyclerView = (RecyclerView) findViewById(R.id.recycler_view);
 
        mAdapter = new MoviesAdapter(movieList);
 
        recyclerView.setHasFixedSize(true);
        RecyclerView.LayoutManager mLayoutManager = new LinearLayoutManager(getApplicationContext());
        recyclerView.setLayoutManager(mLayoutManager);
        recyclerView.addItemDecoration(new DividerItemDecoration(this, LinearLayoutManager.VERTICAL));
        recyclerView.setItemAnimator(new DefaultItemAnimator());
        recyclerView.setAdapter(mAdapter);
 
        recyclerView.addOnItemTouchListener(new RecyclerTouchListener(getApplicationContext(), recyclerView, new ClickListener() {
            @Override
            public void onClick(View view, int position) {
                Movie movie = movieList.get(position);
                Toast.makeText(getApplicationContext(), movie.getTitle() + " is selected!", Toast.LENGTH_SHORT).show();
            }
 
            @Override
            public void onLongClick(View view, int position) {
 
            }
        }));
 
        prepareMovieData();
    }
 
    private void prepareMovieData() {
        Movie movie = new Movie("Mad Max: Fury Road", "Action & Adventure", "2015");
        movieList.add(movie);
 
        movie = new Movie("Inside Out", "Animation, Kids & Family", "2015");
        movieList.add(movie);
 
        movie = new Movie("Star Wars: Episode VII - The Force Awakens", "Action", "2015");
        movieList.add(movie);
 
        movie = new Movie("Shaun the Sheep", "Animation", "2015");
        movieList.add(movie);
 
        movie = new Movie("The Martian", "Science Fiction & Fantasy", "2015");
        movieList.add(movie);
 
        movie = new Movie("Mission: Impossible Rogue Nation", "Action", "2015");
        movieList.add(movie);
 
        movie = new Movie("Up", "Animation", "2009");
        movieList.add(movie);
 
        movie = new Movie("Star Trek", "Science Fiction", "2009");
        movieList.add(movie);
 
        movie = new Movie("The LEGO Movie", "Animation", "2014");
        movieList.add(movie);
 
        movie = new Movie("Iron Man", "Action & Adventure", "2008");
        movieList.add(movie);
 
        movie = new Movie("Aliens", "Science Fiction", "1986");
        movieList.add(movie);
 
        movie = new Movie("Chicken Run", "Animation", "2000");
        movieList.add(movie);
 
        movie = new Movie("Back to the Future", "Science Fiction", "1985");
        movieList.add(movie);
 
        movie = new Movie("Raiders of the Lost Ark", "Action & Adventure", "1981");
        movieList.add(movie);
 
        movie = new Movie("Goldfinger", "Action & Adventure", "1965");
        movieList.add(movie);
 
        movie = new Movie("Guardians of the Galaxy", "Science Fiction & Fantasy", "2014");
        movieList.add(movie);
 
        mAdapter.notifyDataSetChanged();
    }
 
    public interface ClickListener {
        void onClick(View view, int position);
 
        void onLongClick(View view, int position);
    }
 
    public static class RecyclerTouchListener implements RecyclerView.OnItemTouchListener {
 
        private GestureDetector gestureDetector;
        private MainActivity.ClickListener clickListener;
 
        public RecyclerTouchListener(Context context, final RecyclerView recyclerView, final MainActivity.ClickListener clickListener) {
            this.clickListener = clickListener;
            gestureDetector = new GestureDetector(context, new GestureDetector.SimpleOnGestureListener() {
                @Override
                public boolean onSingleTapUp(MotionEvent e) {
                    return true;
                }
 
                @Override
                public void onLongPress(MotionEvent e) {
                    View child = recyclerView.findChildViewUnder(e.getX(), e.getY());
                    if (child != null && clickListener != null) {
                        clickListener.onLongClick(child, recyclerView.getChildPosition(child));
                    }
                }
            });
        }
 
        @Override
        public boolean onInterceptTouchEvent(RecyclerView rv, MotionEvent e) {
 
            View child = rv.findChildViewUnder(e.getX(), e.getY());
            if (child != null && clickListener != null && gestureDetector.onTouchEvent(e)) {
                clickListener.onClick(child, rv.getChildPosition(child));
            }
            return false;
        }
 
        @Override
        public void onTouchEvent(RecyclerView rv, MotionEvent e) {
        }
 
        @Override
        public void onRequestDisallowInterceptTouchEvent(boolean disallowIntercept) {
 
        }
    }
 
}
```
