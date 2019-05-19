# NotePad
This is an AndroidStudio rebuild of google SDK sample NotePad
期中实验 基于NotePad应用做功能扩展


# 1、NoteList中显示条目增加时间戳显示

notelist_item.xml

    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>

NotePadProvider.java

     private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };

dataColumns，viewIDs

    private String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
    private int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

NotePadProvider.java
        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);

NoteEditor.java

        Long now = Long.valueOf(System.currentTimeMillis());
        Date date = new Date(now);
        SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        String dateTime = format.format(date);
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);

运行效果如图所示：

<image src="https://github.com/1158509577/notepad/blob/master/create.png">

# 2、笔记查询功能（根据标题查询）

list_options_menu.xml

    <item
        android:id="@+id/menu_search"
        android:title="@string/menu_search"
        android:icon="@drawable/search"
        android:showAsAction="always">
    </item>

note_search_list.xml 搜索控件SearchView和ListView控件显示搜索结果列表：

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">

        <SearchView
            android:id="@+id/search_view"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:iconifiedByDefault="false"
            android:queryHint="输入搜索内容..."
            android:layout_alignParentTop="true">
        </SearchView>

        <ListView
            android:id="@android:id/list"
            android:layout_width="match_parent"
            android:layout_height="wrap_content">
        </ListView>

    </LinearLayout>

NoteList.java onOptionsItemSelected方法中

    case R.id.menu_search:
         Intent intent = new Intent();
         intent.setClass(NotesList.this,NoteSearch.class);
         NotesList.this.startActivity(intent);
         return true;

NoteSearch.java 使NoteSearch继承ListActivity并实现SearchView.OnQueryTextListener接口
对SearchView文本变化设置监听

    public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {

        private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
                //扩展 显示时间 颜色
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
                NotePad.Notes.COLUMN_NAME_BACK_COLOR
        };

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.note_search_list);
            Intent intent = getIntent();
            if (intent.getData() == null) {
                intent.setData(NotePad.Notes.CONTENT_URI);
            }
            SearchView searchview = (SearchView)findViewById(R.id.search_view);
            searchview.setOnQueryTextListener(NoteSearch.this);  //为查询文本框注册监听器
        }

        @Override
        public boolean onQueryTextSubmit(String query) {
            return false;
        }

        @Override
        public boolean onQueryTextChange(String newText) {

            String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";

            String[] selectionArgs = { "%"+newText+"%" };

            Cursor cursor = managedQuery(
                    getIntent().getData(),            // Use the default content URI for the provider.
                    PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                    selection,                        // 条件左边
                    selectionArgs,                    // 条件右边
                    NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
            );

            String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
            int[] viewIDs = { android.R.id.text1 , R.id.text1_time };

            MyCursorAdapter adapter = new MyCursorAdapter(
                    this,
                    R.layout.noteslist_item,
                    cursor,
                    dataColumns,
                    viewIDs
            );
            setListAdapter(adapter);
            return true;
        }

        @Override
        protected void onListItemClick(ListView l, View v, int position, long id) {

            // Constructs a new URI from the incoming URI and the row ID
            Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);

            // Gets the action from the incoming Intent
            String action = getIntent().getAction();

            // Handles requests for note data
            if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {

                // Sets the result to return to the component that called this Activity. The
                // result contains the new URI
                setResult(RESULT_OK, new Intent().setData(uri));
            } else {

                // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
                // Intent's data is the note ID URI. The effect is to call NoteEdit.
                startActivity(new Intent(Intent.ACTION_EDIT, uri));
            }
        }

    }

点击搜索图标：

<image src="https://github.com/1158509577/notepad/blob/master/save.png">

# 3、UI美化

NotePad.java

    public static final String COLUMN_NAME_BACK_COLOR = "color";
    public static final int DEFAULT_COLOR = 0; //白
    public static final int ANTIQUE_WHITE_COLOR = 1;
    public static final int SKY_BLUE_COLOR = 2;
    public static final int VIOLET_COLOR = 3;
    public static final int PINK_COLOR = 4;
    public static final int GREEN_COLOR = 5;
    public static final int LIGHT_CORAL_COLOR = 6;
    public static final int BLACK_COLOR = 7;

修改创建数据库表语句

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"
                + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER" //颜色数据
                + ");");
    }

自定义一个MyCursorAdapter类继承SimpleCursorAdapter，cursor读取的数据库内容填充到item，将颜色填充：
MyCursorAdapter.java

    public class MyCursorAdapter extends SimpleCursorAdapter {

         public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
                super(context, layout, c, from, to);
         }

         @Override
         public void bindView(View view, Context context, Cursor cursor){
                super.bindView(view, context, cursor);
                //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
                int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));

                switch (x){
                    case NotePad.Notes.DEFAULT_COLOR:
                        view.setBackgroundColor(Color.rgb(255, 255, 255));
                        break;
                    case NotePad.Notes.ANTIQUE_WHITE_COLOR:
                        view.setBackgroundColor(Color.rgb(250, 235, 215));
                        break;
                    case NotePad.Notes.SKY_BLUE_COLOR:
                        view.setBackgroundColor(Color.rgb(154,252,247));
                        break;
                    case NotePad.Notes.VIOLET_COLOR:
                        view.setBackgroundColor(Color.rgb(221, 160, 221));
                        break;
                    case NotePad.Notes.PINK_COLOR:
                        view.setBackgroundColor(Color.rgb(255, 192, 203));
                        break;
                    case NotePad.Notes.GREEN_COLOR:
                        view.setBackgroundColor(Color.rgb(162, 247, 218));
                        break;
                    case NotePad.Notes.LIGHT_CORAL_COLOR:
                        view.setBackgroundColor(Color.rgb(241, 167, 159));
                        break;
                    default:
                        view.setBackgroundColor(Color.rgb(255, 255, 255));
                        break;
                }
         }
    }
    
    
<image src="https://github.com/1158509577/notepad/blob/master/changecolor.png">
    
    
# 4、背景更换

editor_options_menu.xml

    <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/color"
        android:showAsAction="always"/>
 
note_color.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal" android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageButton
            android:id="@+id/color_white"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorWhite"
            android:onClick="white"/>

        <ImageButton
            android:id="@+id/color_antiquewhite"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorAntiqueWhite"
            android:onClick="antique_white"/>

        <ImageButton
            android:id="@+id/color_sky_blue"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorSkyBlue"
            android:onClick="sky_blue"/>

        <ImageButton
            android:id="@+id/color_violet"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorViolet"
            android:onClick="violet"/>

        <ImageButton
            android:id="@+id/color_pink"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorPink"
            android:onClick="pink"/>

        <ImageButton
            android:id="@+id/color_green"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorGreen"
            android:onClick="green"/>

        <ImageButton
            android:id="@+id/color_light_coral"
            android:layout_width="0dp"
            android:layout_height="50dp"
            android:layout_weight="1"
            android:background="@color/colorLightCoral"
            android:onClick="light_coral"/>

    </LinearLayout>

value/color.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="colorWhite">#ffffff</color>
    <color name="colorAntiqueWhite">#faebd7</color>
    <color name="colorSkyBlue">#9afcf7</color>
    <color name="colorViolet">#90DD5A</color>
    <color name="colorPink">#F70028</color>
    <color name="colorGreen">#32D59A</color>
    <color name="colorLightCoral">#692C27</color>
    <color name="colorBlack">#000000</color>
    <color name="colorPrimary">#FF8C00</color>
</resources>

NoteColor.java

    public class NoteColor extends Activity {

        private Cursor mCursor;
        private Uri mUri;
        private int color;
        private static final int COLUMN_INDEX_TITLE = 1;

        private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
        };

        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.note_color);
            mUri = getIntent().getData();
            mCursor = managedQuery(
                    mUri,        // The URI for the note that is to be retrieved.
                    PROJECTION,  // The columns to retrieve
                    null,        // No selection criteria are used, so no where columns are needed.
                    null,        // No where columns are used, so no where values are needed.
                    null         // No sort order is needed.
            );

        }

        @Override
        protected void onResume(){
            if (mCursor != null) {
                mCursor.moveToFirst();
                color = mCursor.getInt(COLUMN_INDEX_TITLE);
            }
            super.onResume();
        }

        @Override
        protected void onPause() {
            super.onPause();
            ContentValues values = new ContentValues();
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
            getContentResolver().update(mUri, values, null, null);

        }

        public void white(View view){
            color = NotePad.Notes.DEFAULT_COLOR;
            finish();
        }

        public void antique_white(View view){
            color = NotePad.Notes.ANTIQUE_WHITE_COLOR;
            finish();
        }

        public void sky_blue(View view){
            color = NotePad.Notes.SKY_BLUE_COLOR;
            finish();
        }
        public void violet(View view){
            color = NotePad.Notes.VIOLET_COLOR;
            finish();
        }

        public void pink(View view){
            color = NotePad.Notes.PINK_COLOR;
            finish();
        }

        public void green(View view){
            color = NotePad.Notes.GREEN_COLOR;
            finish();
        }

        public void light_coral(View view){
            color = NotePad.Notes.LIGHT_CORAL_COLOR;
            finish();
        }

    }
    
NoteEditor.java

    private final void changeColor() {
            Intent intent = new Intent(null,mUri);
            intent.setClass(NoteEditor.this,NoteColor.class);
            NoteEditor.this.startActivity(intent);
    }
    



# 5、导出笔记

editor_options_menu.xml

    <item android:id="@+id/menu_output"
          android:title="@string/menu_export" />
          
export_text.xml

    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:paddingLeft="6dip"
        android:paddingRight="6dip"
        android:paddingBottom="3dip">

        <EditText android:id="@+id/output_name"
            android:maxLines="1"
            android:layout_marginTop="2dp"
            android:layout_marginBottom="15dp"
            android:layout_width="wrap_content"
            android:ems="25"
            android:layout_height="wrap_content"
            android:autoText="true"
            android:capitalize="sentences"
            android:scrollHorizontally="true" />

        <Button android:id="@+id/output_ok"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="right"
            android:text="@string/export_ok"
            android:onClick="OutputOk" />

    </LinearLayout>
    
NoteExport.java

    public class NoteExport extends Activity {

        private static final String[] PROJECTION = new String[] {
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
                NotePad.Notes.COLUMN_NAME_NOTE, // 2
                NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
                NotePad.Notes.COLUMN_NAME_BACK_COLOR, //5
        };

        private String TITLE;
        private String NOTE;
        private String CREATE_DATE;
        private String MODIFICATION_DATE;

        private Cursor mCursor;
        private EditText mName;
        private Uri mUri;

        private boolean flag = false;

        private static final int COLUMN_INDEX_TITLE = 1;

        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.export_text);

            mUri = getIntent().getData();
            mCursor = managedQuery(
                    mUri,        // The URI for the note that is to be retrieved.
                    PROJECTION,  // The columns to retrieve
                    null,        // No selection criteria are used, so no where columns are needed.
                    null,        // No where columns are used, so no where values are needed.
                    null         // No sort order is needed.
            );

            mName = (EditText) findViewById(R.id.output_name);
        }

        @Override
        protected void onResume(){
            super.onResume();
            if (mCursor != null) {
                // The Cursor was just retrieved, so its index is set to one record *before* the first
                // record retrieved. This moves it to the first record.
                mCursor.moveToFirst();
                // Displays the current title text in the EditText object.
                mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
            }
        }

        @Override
        protected void onPause() {
            super.onPause();
            if (mCursor != null) {
                TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
                NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
                CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
                MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
                if (flag == true) {
                    write();
                }
                flag = false;
            }
        }

        public void OutputOk(View v){
            flag = true;
            finish();
        }

        private void write() {
            try {
                // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
                if (Environment.getExternalStorageState().equals(
                        Environment.MEDIA_MOUNTED)) {
                    // 获取SD卡的目录
                    File sdCardDir = Environment.getExternalStorageDirectory();
                    //创建文件目录
                    File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                    //写文件
                    PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                    ps.println(TITLE);
                    ps.println(NOTE);
                    ps.println("创建时间：" + CREATE_DATE);
                    ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                    ps.close();
                    Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt",              Toast.LENGTH_LONG).show();
                }
            }
            catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
NoteEditor.java

    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteExport.class);
        NoteEditor.this.startActivity(intent);
    }
    
<image src="https://github.com/1158509577/notepad/blob/master/export1.png">
    
# 5、笔记排序

list_options_menu.xml

    <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@android:drawable/ic_menu_sort_by_size"
        android:showAsAction="always" >
        <menu>
            <item
                android:id="@+id/menu_sort1"
                android:title="@string/menu_sort1"/>
            <item
                android:id="@+id/menu_sort2"
                android:title="@string/menu_sort2"/>
            <item
                android:id="@+id/menu_sort3"
                android:title="@string/menu_sort3"/>
        </menu>
    </item>

NoteList.java

    //按创建时间排序
    case R.id.menu_sort1:
        cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes._ID  // Use the default sort order.
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;

    //按修改时间排序
    case R.id.menu_sort2:
        cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes.DEFAULT_SORT_ORDER // Use the default sort order.
        );

        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;

    //按颜色排序
    case R.id.menu_sort3:
        cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                null,                             // No where clause, return all records.
                null,                             // No where clause, therefore no where column values.
                NotePad.Notes.COLUMN_NAME_BACK_COLOR // Use the default sort order.
        );
        adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;


<image src="https://github.com/1158509577/notepad/blob/master/sort1.png">
  
<image src="https://github.com/1158509577/notepad/blob/master/sbcolor.png">
  
<image src="https://github.com/1158509577/notepad/blob/master/sbcreate.png">
  
<image src="https://github.com/1158509577/notepad/blob/master/sbmodi.png">
  
  
