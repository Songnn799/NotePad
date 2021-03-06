# 期中实验NotePad
### 一、成果展示
#####  1.点进首页，即可看到添加了时间戳
![首页-时间戳](https://note.youdao.com/yws/api/personal/file/BF99CA3D273A47F4B491B706CCDD69B1?method=download&shareKey=68dc13edb434bd891f7736b03e4f4c43)
#####  2.点击搜索按钮，就能展开搜索框并弹出键盘。
![搜索](https://note.youdao.com/yws/api/personal/file/0B418ACD575A4944BA995242F1E8BD1E?method=download&shareKey=a61bdf7406a74df90075b5e48a684a42)
#####  3.在输入内容时，会匹配可能的搜索结果，点击相关条目就能进入编辑界面。
![匹配搜索结果](https://note.youdao.com/yws/api/personal/file/62A4C480259C4130BB2A11C41C23F787?method=download&shareKey=63853ee434a2861290a8f88efa43728f)
#####  4.扩展功能1：在撰写笔记时，可修改笔记字体的大小及颜色
![扩展功能1-菜单](https://note.youdao.com/yws/api/personal/file/BD524799925A483A91627E34CAE6F9A4?method=download&shareKey=07ace68160209735743c23d93c3b4158)
![扩展功能1-字体颜色](https://note.youdao.com/yws/api/personal/file/3FB17FB2336B402C9AD509469674C146?method=download&shareKey=b33c4f0f21b910bb6a90435915783f9e)
![扩展功能1-字体大小](https://note.youdao.com/yws/api/personal/file/4A335CC6545B46EA8AF04B59B26BAEF5?method=download&shareKey=f45c54b3c1e06c28ebd2f964e8933adc)
#####  4.扩展功能2：修改背景颜色
![扩展功能2-修改背景](https://note.youdao.com/yws/api/personal/file/C50347BCAE474FCE9839A17F3DE5977F?method=download&shareKey=92168fd3d7d4d8074cb5f49b5c09b78f)
### 二、基础功能：“时间戳”功能
##### 1.修改样式：打开note_list.xml，在原来的TextView下方新增一个用来显示日期的TextView：
```xml
 <RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
        />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
        />
</RelativeLayout>
```
##### 2.在NotePadProvider类中找到insert()方法，首先将系统默认显示时间（一堆数字，很难读）修改为易读的时间格式，增加可读性：
```java
        //系统默认显示的时间使用毫秒数进行表示，很难阅读
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        //将时间格式化为常见的年月日表示形式
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
        String dateFormat = simpleDateFormat.format(date);
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
        }
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
        }
```
##### 3.到NoteList中将时间参数添加进去：
```java
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//日期
    };
```
##### 4.在NoteEditor中的updateNote()方法中的默认的毫秒数时间格式对其进行时间的格式化，修改代码如下：
```java
long now = System.currentTimeMillis();
    Date date = new Date(now);
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
    simpleDateFormat.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
    String dateFormat = simpleDateFormat.format(date);
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
```
##### 5.查看源码发现NoteList中只是显示id以及titile，并没有时间，因此我们加上时间的显示，而后装载相应数据列，修改代码如下：
```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//日期
    };
    private static final int COLUMN_INDEX_TITLE = 1;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setDefaultKeyMode(DEFAULT_KEYS_SHORTCUT);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        getListView().setOnCreateContextMenuListener(this);
        Cursor cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            null,
            null,
            NotePad.Notes.DEFAULT_SORT_ORDER
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
        int[] viewIDs = { android.R.id.text1, R.id.text2 }
        SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
                      this,
                      R.layout.noteslist_item,
                      cursor,
                      dataColumns,
                      viewIDs
              );
        setListAdapter(adapter);
    }

String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
int[] viewIDs = { android.R.id.text1, R.id.text2 };
```
### 三、基础功能：搜索功能
##### 1.首先在list_option_menu.xml，添加一个搜索图标：
```xml
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
```
##### 2.在NoteList类的onOptionsItemSelected方法中添加menu_search图标的点击选择事件：
```java
    case R.id.menu_search:
        Intent intent = new Intent();
        intent.setClass(this, NoteSearch.class);
        this.startActivity(intent);
        return true;
```
##### 3.新建NoteSearch的activity（添加NoteSearch活动）：
```java
public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;//
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 修改时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);//设置全屏显示
        super.setContentView(R.layout.note_search);
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
        search.setOnQueryTextListener(NoteSearch.this);
        listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
                //为每个item添加点击事件，点击可以查看笔记具体内容
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                String action = getIntent().getAction();
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return true;
    }
    @Override
    public boolean onQueryTextChange(String newText) {//实现模糊查询，通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { android.R.id.text1,android.R.id.text2};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,
                R.layout.noteslist_item,
                cursor,
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
}
```
##### 4.新建note_search.xml（NoteSearch的布局文件）：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索">
    </SearchView>
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```
##### 5.在AndroidManifest中添加search的启动器：
```xml
    <activity android:name=".NoteSearch" android:label="@string/title_notes_list">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
    </activity>
```
### 四、扩展功能一：修改笔记中字体的大小及颜色
##### 1.将字体大小和字体颜色选项添加到菜单里(在editor_options_menu.xml中添加如下代码）
```xml
<item
    android:id="@+id/font_size"
    android:title="Font Size">
    <menu>
        <group>
            <item
                android:id="@+id/font_10"
                android:title="font10"/>
            <item
                android:id="@+id/font_16"
                android:title="font16" />
            <item
                android:id="@+id/font_20"
                android:title="font20" />
        </group>
    </menu>
</item>

<item
    android:title="Font Color"
    android:id="@+id/font_color">
    <menu>
        <group>
            <item
                android:title="red"
                android:id="@+id/red_font"/>
            <item
                android:title="blue"
                android:id="@+id/blue_font"/>
            <item
                android:title="black"
                android:id="@+id/black_font"/>
        </group>
    </menu>
</item>
```
##### 2.在NoteEditor的onOptionsItemSelected中添加相应的case来选择相应事件
```java
case R.id.font_10:
    mText.setTextSize(20);
    Toast toast =Toast.makeText(NoteEditor.this,"修改成功", Toast.LENGTH_SHORT);
    toast.show();
    finish();
    break;
case R.id.font_16:
    mText.setTextSize(32);
    Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast2.show();
    finish();
    break;
case R.id.font_20:
    mText.setTextSize(40);
    Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast3.show();
    finish();
    break;
case R.id.red_font:
    mText.setTextColor(Color.RED);
    Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast4.show();
    finish();
    break;
case R.id.blue_font:
    mText.setTextColor(Color.BLUE);
    Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast5.show();
    finish();
    break;
case R.id.black_font:
    mText.setTextColor(Color.BLACK);
    Toast toast6 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast6.show();
    finish();
    break;
```
##### 3.在NotePad类中添加颜色大小属性定义
```java
 public static final String COLUMN_NAME_TEXT_COLOR = "font_size";
 public static final String COLUMN_NAME_TEXT_SIZE = "font_color";
```
##### 4.在NotePadProvider的onCreate方法中加入相应的参数来保存字体大小和颜色的信息
```java
+ NotePad.Notes.COLUMN_NAME_TEXT_COLOR + "INTEGER"
+ NotePad.Notes.COLUMN_NAME_TEXT_SIZE + "INTEGER" 
```
### 五、扩展功能二：修改背景
##### 在notelist_items中添加语句：
```xml
android:background="@drawable/beijing"
```
