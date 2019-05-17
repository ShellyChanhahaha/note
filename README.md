# Note便签本应用

#### 												——Android期中作业

## 基本功能实现

### 1.增加时间戳

#### 效果截图

![时间戳.png](https://i.loli.net/2019/05/17/5cde0c440344259008.png)



#### 关键代码

##### 将数据库的笔记的修改时间字段改成字符类型

在NotePadProvider.java文件中修改数据库表的创建，将NotePad.Notes.COLUMN_NAME_CREATE_DATE 和NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE的类型修改成 “ TEXT”

```java
/**
 *
 * Creates the underlying database with table name and column names taken from the
 * NotePad class.
 */
@Override
public void onCreate(SQLiteDatabase db) {
    db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_PICTURE_PATH + " TEXT"
            + ");");
}
```



##### 修改NotePadProvider.java中的insert方法

添加时间格式，以及修改时区，如果不修改时区的话，时间会早8个小时

```java
// Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());
//定义显示的格式
SimpleDateFormat dateformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
//将时区设置为GMT+8
dateformat.setTimeZone(TimeZone.getTimeZone("GMT+8"));
//格式化获取的时间
String time = dateformat.format(now);
```

##### 在NoteEditor的updateNote函数中也加上对时间的转格式。跟上面的一模一样，这里不赘述了

##### 在noteslist_item.xml中增加一个TextView布局，用来放时间

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    >
    <TextView
        android:id="@android:id/text1"
        android:layout_width="wrap_content"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="top"
        android:paddingLeft="5dip"
        />
    <TextView
        android:id="@android:id/text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:gravity="bottom"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
</LinearLayout>
```

##### 在NotesList.java中修改查询的投影

添加NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE

```java
/**
 * The columns needed by the cursor adapter
 */
private static final String[] PROJECTION = new String[]{
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE //2
};
```

##### 最后在setAdapter里面将listview设置进去就好啦！



### 2.增加搜索框

#### 效果截图

![搜索.png](https://i.loli.net/2019/05/17/5cde0ca6bc69076102.png)

输入可进行实时对标题的模糊搜索

![搜索1.png](https://i.loli.net/2019/05/17/5cde0cb5ca80c27465.png)

搜索失败会提示

![搜索3.png](https://i.loli.net/2019/05/17/5cde0cca1e74727293.png)

#### 关键代码

##### 在原本放listview的布局文件中添加SearchView

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <SearchView
        android:id="@+id/searchview"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginLeft="10dp"
        android:layout_marginRight="10dp"
        android:layout_marginTop="10dp"
        android:layout_marginBottom="10dp"
        android:background="@drawable/searchview"
        >
    </SearchView>

    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">
    </ListView>

</LinearLayout>
```

##### 在NotesList.java中设置SearchView的属性以及监听

```java
 searchView = (SearchView) findViewById(R.id.searchview);
       searchView.setIconifiedByDefault(true);
       searchView.setSubmitButtonEnabled(true);
       searchView.setQueryHint("搜一搜");
       searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
           @Override
           public boolean onQueryTextSubmit(String s) {
               String selection = NotePad.Notes.COLUMN_NAME_TITLE + " GLOB '*" + s + "*'";

               cursor = getContentResolver().query(
                       getIntent().getData(),            // Use the default content URI for the provider.
                       PROJECTION,                       // Return the note ID and title for each note.
                       selection,                             // No where clause, return all records.
                       null,                             // No where clause, therefore no where column values.
                       NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.

               );
               if (cursor.moveToNext()) {
                   adapter = new SimpleCursorAdapter(
                           NotesList.this,                             // The Context for the ListView
                           R.layout.noteslist_item,          // Points to the XML for a list item
                           cursor,                           // The cursor to get items from
                           dataColumns,
                           viewIDs
                   );
                   listView.setAdapter(adapter);
               } else {
                   Toast.makeText(NotesList.this, "no such note!", Toast.LENGTH_SHORT).show();
               }
               return true;
           }

           @Override
           public boolean onQueryTextChange(String s) {
               String selection = NotePad.Notes.COLUMN_NAME_TITLE + " GLOB '*" + s + "*'";

               cursor = getContentResolver().query(
                       getIntent().getData(),            // Use the default content URI for the provider.
                       PROJECTION,                       // Return the note ID and title for each note.
                       selection,                             // No where clause, return all records.
                       null,                             // No where clause, therefore no where column values.
                       NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.

               );

               if (cursor.moveToNext())

                   Log.i("daawdwad", selection);
/*                else {
                  cursor = getContentResolver().query(
                           getIntent().getData(),            // Use the default content URI for the provider.
                           PROJECTION,                       // Return the note ID and title for each note.
                           null,                             // No where clause, return all records.
                           null,                             // No where clause, therefore no where column values.
                           NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.

                   );
                   Toast.makeText(NotesList.this,"no such note!",Toast.LENGTH_SHORT).show();
               }*/

               adapter.swapCursor(cursor);
               return false;
           }
       });
```



## 拓展功能

### 1.美化UI

#### 1)美化UI之更换主题

#### 效果截图

![选择主题1.png](https://i.loli.net/2019/05/17/5cde0dc1cbf8e50698.png)

点击设置主题风格

![选择主题2.png](https://i.loli.net/2019/05/17/5cde0df55d1a661254.png)

选择“老夫的少女心”

![选择主题pink.png](https://i.loli.net/2019/05/17/5cde0df57479749470.png)

选择“撒哈拉的故事”

![选择主题yellow.png](https://i.loli.net/2019/05/17/5cde0fcd8567c76422.png)



#### 关键代码

##### 在list_options_menu.xml中添加设置主题的选项

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:textColor="@color/black"
    android:background="@color/white">
    <!--  This is our one standard application action (creating a new note). -->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          app:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          so that the user can paste in the data.. -->
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p' />
        <item android:id="@+id/bg_theme"
              android:title="@string/style">
            <menu>
                <group>
                    <item
                        android:id="@+id/blue"
                        android:title="海边的卡夫卡"/>
                    <item
                        android:id="@+id/pink"
                        android:title="@string/theme_pink"/>
                    <item
                        android:id="@+id/purple"
                        android:title="@string/theme_purple"/>
                    <item
                        android:id="@+id/yellow"
                        android:title="@string/theme_yellow"/>
                    <item
                        android:id="@+id/green"
                        android:title="@string/theme_green"/>
                    <item
                        android:id="@+id/theme_default"
                        android:title="@string/theme_default"/>
            </group>
        </menu>
    </item>
</menu>
```



##### 在NotesList.java中添加对新增的选项菜单的监听

使用SharedPreferences存储当前的样式设置

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    SharedPreferences setting = getSharedPreferences(PRE_NAME, 0);
    SharedPreferences.Editor edit = setting.edit();
    switch (item.getItemId()) {
        case R.id.menu_add:
            /*
             * Launches a new Activity using an Intent. The intent filter for the Activity
             * has to have action ACTION_INSERT. No category is set, so DEFAULT is assumed.
             * In effect, this starts the NoteEditor Activity in NotePad.
             */
            startActivity(new Intent(Intent.ACTION_INSERT, getIntent().getData()));
            return true;
        case R.id.menu_paste:
            /*
             * Launches a new Activity using an Intent. The intent filter for the Activity
             * has to have action ACTION_PASTE. No category is set, so DEFAULT is assumed.
             * In effect, this starts the NoteEditor Activity in NotePad.
             */
            startActivity(new Intent(Intent.ACTION_PASTE, getIntent().getData()));
            return true;
        case R.id.blue:
            themeID = R.style.blueTheme;
            edit.putInt("thememode", themeID);
            edit.commit();
            break;
        //recreate();
        //return true;
        case R.id.pink:
            themeID = R.style.pinkTheme;
            edit.putInt("thememode", themeID);
            edit.commit();
            break;
        //recreate();
        //return true;
        case R.id.green:
            themeID = R.style.greenTheme;
            edit.putInt("thememode", themeID);
            edit.commit();
            break;
        //recreate();
        //return true;
        case R.id.purple:
            themeID = R.style.purpleTheme;
            edit.putInt("thememode", themeID);
            edit.commit();
            break;

        case R.id.yellow:
            themeID = R.style.redTheme;
            edit.putInt("thememode", themeID);
            edit.commit();
            break;

        case R.id. theme_default:
            themeID=R.style.AppTheme;
            edit.putInt("thememode",themeID);
            edit.commit();
            break;

        default:
            return super.onOptionsItemSelected(item);
    }
    recreate();
    return true;
}
```

同时，在NotesList.java的开头判断SharedPreferences里thememode的值，如果有值，则设定该主题，如果没有则设定默认主题

```java
SharedPreferences setting = getSharedPreferences(PRE_NAME, 0);
themeID = setting.getInt("thememode", R.style.AppTheme);
setTheme(themeID);
```

##### 在styles.xml中添加自定义的样式

```xml
<resources>

    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="pinkTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/lightpink</item>
        <item name="android:textColorPrimary">@color/white</item>
        <item name="colorPrimaryDark">@color/black</item>
        <item name="colorAccent">@color/Honeydew</item>
        <!--窗体的背景色-->
        <item name="android:windowBackground">@color/MistyRose</item>
        <item name="android:textColor">@color/black</item>
    </style>
    <style name="greenTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/LightGreen</item>
        <item name="android:textColorPrimary">@color/white</item>
        <item name="colorPrimaryDark">@color/black</item>
        <item name="colorAccent">@color/Honeydew</item>
        <!--窗体的背景色-->
        <item name="android:windowBackground">@color/Honeydew</item>
        <item name="android:textColor">@color/black</item>
    </style>
    <style name="purpleTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/darkMagenta</item>
        <item name="android:textColorPrimary">@color/black</item>
        <item name="colorPrimaryDark">@color/black</item>
        <item name="colorAccent">@color/plum</item>
        <!--窗体的背景色-->
        <item name="android:windowBackground">@color/thistle</item>
        <item name="android:textColor">@color/black</item>
    </style>
    <style name="blueTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/LightBLue</item>
        <item name="android:textColorPrimary">@color/black</item>
        <item name="colorPrimaryDark">@color/black</item>
        <item name="colorAccent">@color/AliceBlue</item>
        <!--窗体的背景色-->
        <item name="android:windowBackground">@color/AliceBlue</item>
        <item name="android:textColor">@color/black</item>
    </style>

    <style name="redTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/Gold</item>
        <item name="android:textColorPrimary">@color/black</item>
        <item name="colorPrimaryDark">@color/black</item>
        <item name="colorAccent">@color/LemonChiffon</item>
        <!--窗体的背景色-->
        <item name="android:windowBackground">@color/LemonChiffon</item>
        <item name="android:textColor">@color/black</item>
    </style>
</resources>
```



#### 2)美化UI之更换信纸

#### 效果截图

![选择信纸.png](https://i.loli.net/2019/05/17/5cde0eec6859066164.png)

![选择信纸2.png](https://i.loli.net/2019/05/17/5cde0eec3959491935.png)

![选择信纸3.png](https://i.loli.net/2019/05/17/5cde0eec70bfc26514.png)

#### 关键代码

##### 创建选择信纸的布局文件——以对话框形式弹出

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="40dp">
        <TextView
            android:layout_width="match_parent"
            android:layout_height="match_parent" 
            android:text="@string/change_paper"
            android:id="@+id/change_paper_text"
            android:gravity="center"
            />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:orientation="horizontal"
        android:clickable="true">

        <ImageView
            android:id="@+id/paper1"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@drawable/paper1"
            android:clickable="true"
            android:layout_weight="1"/>

        <ImageView
            android:id="@+id/paper2"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@drawable/paper2"
            android:clickable="true"
            android:layout_weight="1"/>

        <ImageView
            android:id="@+id/paper3"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@drawable/paper3"
            android:clickable="true"
            android:layout_weight="1"/>

        <ImageView
            android:id="@+id/paper4"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@drawable/paper4"
            android:clickable="true"
            android:layout_weight="1"/>

        <ImageView
            android:id="@+id/paper5"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@drawable/paper5"
            android:clickable="true"
            android:layout_weight="1"/>
    </LinearLayout>
</LinearLayout>
```

##### 在NotesEditor的选项菜单中添加选择信纸的选项菜单——choose_paper

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item android:id="@+id/menu_save"
          android:icon="@drawable/ic_menu_save"
          android:alphabeticShortcut='s'
          android:title="@string/menu_save"
          app:showAsAction="ifRoom|withText" />
    <item android:id="@+id/menu_revert"
          android:icon="@drawable/ic_menu_revert"
          android:title="@string/menu_revert" />
    <item android:id="@+id/menu_delete"
          android:icon="@drawable/ic_menu_delete"
          android:title="@string/menu_delete"
          app:showAsAction="ifRoom|withText" />
    <item android:id="@+id/menu_photo"
        android:icon="@drawable/ic_menu_media"
        android:title="@string/menu_photo"
        app:showAsAction="ifRoom|withText"/>

    <item android:id="@+id/menu_change_paper"
        android:title="@string/change_paper"/>
</menu>
```

##### 在NotesEditor中选项菜单的监听中添加对选择信纸菜单的监听操作

```java
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    // Handle all of the possible menu actions.
    switch (item.getItemId()) {
        case R.id.menu_save:
            String text = mText.getText().toString();
            updateNote(text, null,pictureFile+"");
            finish();
            break;
        case R.id.menu_delete:
            deleteNote();
            finish();
            break;
        case R.id.menu_revert:
            cancelNote();
            break;
        case R.id.menu_photo:
            photo();
            break;
        case R.id.menu_change_paper:
            changePaper();
            break;
    }
    return super.onOptionsItemSelected(item);
}
/**
 * 实现更换信纸功能
 */
private final void changePaper(){
    final AlertDialog alertDialog = new AlertDialog.Builder(NoteEditor.this).create();
    alertDialog.show();
    Window window = alertDialog.getWindow();
    window.setContentView(R.layout.choose_paper);

    ImageView paper1=(ImageView)  alertDialog.getWindow().findViewById(R.id.paper1);
    ImageView paper2=(ImageView)  alertDialog.getWindow().findViewById(R.id.paper2);
    ImageView paper3=(ImageView)  alertDialog.getWindow().findViewById(R.id.paper3);
    ImageView paper4=(ImageView)  alertDialog.getWindow().findViewById(R.id.paper4);
    ImageView paper5=(ImageView)  alertDialog.getWindow().findViewById(R.id.paper5);

    paper1.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            backgroundPaper=R.drawable.paper1;
            getWindow().setBackgroundDrawableResource(backgroundPaper);
            SharedPreferences settings=getSharedPreferences(PRE_NAME,MODE_PRIVATE);
            SharedPreferences.Editor editor= settings.edit();
            editor.putInt("backgroundPaper",backgroundPaper);
            editor.commit();
            alertDialog.dismiss();
        }
    });

    paper2.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            backgroundPaper=R.drawable.paper2;
            getWindow().setBackgroundDrawableResource(backgroundPaper);
            SharedPreferences settings=getSharedPreferences(PRE_NAME,MODE_PRIVATE);
            SharedPreferences.Editor editor= settings.edit();
            editor.putInt("backgroundPaper",backgroundPaper);
            editor.commit();
            alertDialog.dismiss();
        }
    });

    paper3.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            backgroundPaper=R.drawable.paper3;
            getWindow().setBackgroundDrawableResource(backgroundPaper);
            SharedPreferences settings=getSharedPreferences(PRE_NAME,MODE_PRIVATE);
            SharedPreferences.Editor editor= settings.edit();
            editor.putInt("backgroundPaper",backgroundPaper);
            editor.commit();
            alertDialog.dismiss();
        }
    });

    paper4.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            backgroundPaper=R.drawable.paper4;
            getWindow().setBackgroundDrawableResource(backgroundPaper);
            SharedPreferences settings=getSharedPreferences(PRE_NAME,MODE_PRIVATE);
            SharedPreferences.Editor editor= settings.edit();
            editor.putInt("backgroundPaper",backgroundPaper);
            editor.commit();
            alertDialog.dismiss();
        }
    });

    paper5.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            backgroundPaper=R.drawable.paper5;
            getWindow().setBackgroundDrawableResource(backgroundPaper);
            SharedPreferences settings=getSharedPreferences(PRE_NAME,MODE_PRIVATE);
            SharedPreferences.Editor editor= settings.edit();
            editor.putInt("backgroundPaper",backgroundPaper);
            editor.commit();
            alertDialog.dismiss();
        }
    });

}
```

### 2.添加闹钟功能

#### 效果截图

![设置闹钟.png](https://i.loli.net/2019/05/17/5cde102013ff168327.png)

![设置闹钟1.png](https://i.loli.net/2019/05/17/5cde102011e1b34122.png)

到时间了，屏幕提示，同时音乐响起

![设置闹钟2.png](https://i.loli.net/2019/05/17/5cde10200fe9723731.png)

#### 关键代码

##### 在上下文菜单中添加“闹钟”的选项

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:id="@+id/context_open"
          android:title="@string/menu_open" />
    <item android:id="@+id/context_copy"
          android:title="@string/menu_copy" />
    <item android:id="@+id/context_delete"
          android:title="@string/menu_delete" />
    <item android:id="@+id/context_alarm"
        android:title="@string/menu_alarm"/>
</menu>
```

##### 在NotesList菜单监听中添加对闹钟选项的监听

```java
case R.id.context_alarm:
    alarm(itemContent);
    // Returns to the caller and skips further processing.
    return true;
```

```java
/**
 * 设置闹钟
 */
private final void alarm(String itemContent) {
    Calendar currentTime = Calendar.getInstance();
    new TimePickerDialog(NotesList.this, 0,
            new TimePickerDialog.OnTimeSetListener() {
                @Override
                public void onTimeSet(TimePicker view,
                                      int hourOfDay, int minute) {
                    Intent alarmIntent = new Intent(NotesList.this, AlarmActivity.class);
                    //alarmIntent.putExtra(AlarmActivity.ITEM_CONTENT,itemContent);
                    PendingIntent pi = PendingIntent.getActivity(NotesList.this, 0, alarmIntent, 0);
                    Calendar c = Calendar.getInstance();
                    c.setTimeInMillis(System.currentTimeMillis());
                    c.set(Calendar.HOUR, hourOfDay);
                    c.set(Calendar.MINUTE, minute);
                    //启动Activity
                    AlarmManager alarmManager = (AlarmManager) getSystemService(ALARM_SERVICE);
                    alarmManager.set(AlarmManager.RTC_WAKEUP, c.getTimeInMillis(), pi);
                    Log.e("HEHE", c.getTimeInMillis() + "");
                    Toast.makeText(NotesList.this, "闹钟设置完毕~", Toast.LENGTH_SHORT).show();
                }
            }, currentTime.get(Calendar.HOUR_OF_DAY), currentTime.get(Calendar.MINUTE), false).show();
}
```

##### 增加一个闹钟响的activity

```java
public class AlarmActivity extends AppCompatActivity {

    public static String ITEM_CONTENT;
    MediaPlayer alarmMusic;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        alarmMusic=MediaPlayer.create(AlarmActivity.this,R.raw.alarm);
        alarmMusic.setLooping(true);
        alarmMusic.start();
        new AlertDialog.Builder(AlarmActivity.this)
                .setTitle("闹钟").
                setMessage("闹钟响了").
                setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        alarmMusic.stop();
                        AlarmActivity.this.finish();
                    }
                }).show();
    }
}
```



### 3.拍照功能

#### 效果截图

![拍照.png](https://i.loli.net/2019/05/17/5cde11320037776895.png)

点击拍照

![拍照2.png](https://i.loli.net/2019/05/17/5cde113205d2e96447.png)

拍照后点击“确认”

![拍照3.png](https://i.loli.net/2019/05/17/5cde113202ac763345.png)

#### 关键代码

##### 在编辑笔记的activity中的菜单选项里添加拍照选项

```xml
<item android:id="@+id/menu_change_paper"
    android:title="@string/change_paper"/>
```

##### 在NoteEditor中添加对该选项菜单的监听

```java
case R.id.menu_photo:
    photo();
    break;
```

```java
/**
 * 实现拍照
 */
private final void photo(){
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) != PackageManager.PERMISSION_GRANTED) {
        ActivityCompat.requestPermissions( this, new String[] { Manifest.permission.CAMERA }, TAKE_PHOTO);
    }
    else{
        pictureFile=new File(Environment.getExternalStorageDirectory(),getTimeStame()+".jpg");
        imageUri=Uri.fromFile(pictureFile);
        Intent intent;

        intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.setAction(MediaStore.ACTION_IMAGE_CAPTURE);
        intent.putExtra(MediaStore.EXTRA_OUTPUT,imageUri);
        startActivityForResult(intent, TAKE_PHOTO);
    }

}
```

##### 由于用的是startActivityForResult，所以要写一个onActivityResult

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    ContentResolver resolver = getContentResolver();
    super.onActivityResult(requestCode, resultCode, data);

    switch (requestCode) {
        case TAKE_PHOTO:
            if (resultCode == RESULT_OK) {
                Bitmap originalBitmap1=null;
                //判断图片是否存在
                try{
                    originalBitmap1=BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                }catch (FileNotFoundException e){
                    e.printStackTrace();
                }
                if(originalBitmap1 != null){//如果图片存在保存URI
                    //将选择的图片追加到EditText中光标所在位置
                    mImage.setVisibility(View.VISIBLE);
                    mImage.setImageBitmap(originalBitmap1);
                    updateNotePicturePath(imageUri.toString());

                }else{
                    Toast.makeText(NoteEditor.this, "获取图片失败", Toast.LENGTH_SHORT).show();
                }

            } else {
                Log.e("result", "is not ok" + resultCode);
            }
            break;

    }
}
```

##### 图片缩放的函数

```java
        /**

         * 图片缩放

         * @param originalBitmap 原始的Bitmap

         * @param newWidth 自定义宽度

         * @return 缩放后的Bitmap

         */

        private Bitmap resizeImage(Bitmap originalBitmap, int newWidth, int newHeight){

            int width = originalBitmap.getWidth();

            int height = originalBitmap.getHeight();

            //定义欲转换成的宽、高

//            int newWidth = 200;

//            int newHeight = 200;

            //计算宽、高缩放率

            float scanleWidth = (float)newWidth/width;

            float scanleHeight = (float)newHeight/height;

            //创建操作图片用的matrix对象 Matrix

            Matrix matrix = new Matrix();

            // 缩放图片动作

            matrix.postScale(scanleWidth,scanleHeight);

            //旋转图片 动作

            //matrix.postRotate(45);

            // 创建新的图片Bitmap

            Bitmap resizedBitmap = Bitmap.createBitmap(originalBitmap,0,0,width,height,matrix,true);

            return resizedBitmap;

        }
```

##### 由于使用的模拟器api为28，所以要动态申请权限

```java
/**
 * 获取权限
 */
public void requestPower() {
    //判断是否已经赋予权限
    if (ContextCompat.checkSelfPermission(this,
            Manifest.permission.WRITE_EXTERNAL_STORAGE)
            != PackageManager.PERMISSION_GRANTED) {
        //如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 true。
        if (ActivityCompat.shouldShowRequestPermissionRationale(this,
                Manifest.permission.WRITE_EXTERNAL_STORAGE)) {//这里可以写个对话框之类的项向用户解释为什么要申请权限，并在对话框的确认键后续再次申请权限
        } else {
            //申请权限，字符串数组内是一个或多个要申请的权限，1是申请权限结果的返回参数，在onRequestPermissionsResult可以得知申请结果
            ActivityCompat.requestPermissions(this,
                    new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE,}, 1);
        }
    }
}
```

##### 在onCreate中添加

```java
// android 7.0系统解决拍照的问题
StrictMode.VmPolicy.Builder builder = new StrictMode.VmPolicy.Builder();
StrictMode.setVmPolicy(builder.build());
builder.detectFileUriExposure();
requestPower();
```

##### 在AndroidManifest.xml中添加

```xml
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```