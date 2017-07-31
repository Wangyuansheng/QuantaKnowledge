**

关于ListView控件的适配器Adapter
-----------------------

**

  本人也只是刚刚接触安卓开发，就想对用的比较多的ListView开始进行一点知识的整理。就拿自己写的一个消息列表来作为例子来整理一波吧。

  在主活动的布局文件中就只需要加入一个ListView的控件。

    <ListView
        android:layout_marginTop="5dp"
        android:dividerHeight="5dp"
        android:divider="#99d9db"
        android:id="@+id/lv_msg"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </ListView>
  
  这里也没什么好讲的...

 然后我们再来定义一波ListView中的item布局，我在这里只是简单地设置了一个头像的ImageView、一个用户名的TextView和一条内容的TextViews并为之一一设置ID。

    <LinearLayout
	    android:layout_marginTop="5dp"
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="horizontal"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent">
	
	    <LinearLayout
	        android:gravity="center"
	        android:layout_weight="1"
	        android:layout_width="0dp"
	        android:layout_height="60dp">
	
	        <ImageView
	            android:id="@+id/iv_head"
	            android:layout_width="65dp"
	            android:layout_height="55dp" />
	
	    </LinearLayout>
	
	    <LinearLayout
	        android:orientation="vertical"
	        android:layout_weight="4"
	        android:layout_width="0dp"
	        android:layout_height="60dp">
	
	        <TextView
	            android:id="@+id/tv_name"
	            android:layout_marginTop="5dp"
	            android:layout_marginLeft="5dp"
	            android:layout_width="100dp"
	            android:layout_height="20dp" />
	
	        <TextView
	            android:id="@+id/tv_msg"
	            android:maxLines="1"
	            android:layout_marginTop="2dp"
	            android:layout_marginLeft="5dp"
	            android:layout_width="match_parent"
	            android:layout_height="30dp" />
	
	    </LinearLayout>

	</LinearLayout>
  
  接下来就是重头戏了——一个自定义的**适配器**。

  这也是我今天挺想去总结的一个点，因为我个人觉得适配器就是一个ListView的灵魂，操控着它的数据更新、操控着它的视图展示等等。

  常用的适配器有BaseAdapter，ArrayAdapter，SimpleAdapter等等。而SimpleAdapter在三者中又用的是比较少的，因为SimpleAdapter已经在内部被写好，改动的地方比较少，因此灵活性较低，不宜被继承改动。而ArrayAdapter则是由BaseAdapter派生而来，具有BaseAdapter的全部功能，再加上继承自ArrayAdapter的适配器在实例化时可直接使用泛型构造，从而使自定义适配器更加灵活。而在本例中的适配器继承的便是ArrayAdapter（因为使用了自定义的一个Msg类。）。


	private int resourceId;
    private List<Msg> message = new ArrayList<Msg>();

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource) {
        super(context, resource);
        resourceId = resource;
    }

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @NonNull List<Msg> objects) {
        super(context, resource, objects);
        resourceId = resource;
        this.message = objects;
    }

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @IdRes int textViewResourceId) {
        super(context, resource, textViewResourceId);
    }

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @NonNull Msg[] objects) {
        super(context, resource, objects);
    }

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @IdRes int textViewResourceId, @NonNull Msg[] objects) {
        super(context, resource, textViewResourceId, objects);
    }

    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @IdRes int textViewResourceId, @NonNull List<Msg> objects) {
        super(context, resource, textViewResourceId, objects);
    }
	


  可见，在继承了ArrayAdater<T>后，IDE会提示你要创建构造方法，而该适配器有6种构造方法。其中的几个参数如context指依附的活动，resource指加载item的布局id，List<T> object则指列表中的对象等等就不一一赘述了，毕竟见名知意。


  而在自定义的adapter中还可以重写内部方法。

  
（一）**public View getView(int position, View convertView, ViewGroup parent)**
  
  此方法实现了加载每一个item的布局的功能，最终返回的是一个View对象，通常与类ViewHolder连用。此类只定义item的布局中存在的控件，以方便在加载整个列表布局时，更加快速地从类里取出存在的item布局。即若某个item的布局若存在于ViewHolder，则不需再重新实例一个新的对象，重新加载该item布局，而是从ViewHolder中取出该item的布局，直接展示在ui层。

	public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
        View item;
        ViewHolder mViewHolder;
        //获取当前位置的position以填充布局
        Msg msg = getItem(position);
        //如果当前view为空，则将当前view填充入viewholder中，否则从viewholder中回调出来
        if (convertView == null) {
            //将当前view加载出来
            item = LayoutInflater.from(getContext()).inflate(resourceId, null);
            mViewHolder = new ViewHolder();
            mViewHolder.head = (ImageView) item.findViewById(R.id.iv_head);
            mViewHolder.name = (TextView) item.findViewById(R.id.tv_name);
            mViewHolder.msg = (TextView) item.findViewById(R.id.tv_msg);
            item.setTag(mViewHolder);
        } else {
            item = convertView;
            mViewHolder = (ViewHolder) item.getTag();
        }

        mViewHolder.head.setImageResource(msg.getImageResource());//设定头像
        mViewHolder.name.setText(msg.getName());//设定用户名
        mViewHolder.msg.setText(msg.getMsg());//设定消息

        return item;
    }

  
（二）**public int getCount()、public int getPosition(T item)、public T getItem(int position)**

  这些方法分别是获取整个列表的长度，即item的数目，一般返回的是mList.size()；获取当前item的位置，返回的是item在列表的位置；获取当前位置的item，返回的是position中对应的item对象。

    @Override
    public int getCount() {
        return message.size();
    }
    
    //获取item的位置

    @Override
    public int getPosition(@Nullable Msg item) {
        return super.getPosition(item);
    }
    
    //获取当前位置的item

    @Nullable
    @Override
    public Msg getItem(int position) {
        return super.getItem(position);
    }

（三）**public void notifyDataSetChanged()**

  这个方法是用来刷新列表的。比如当你需要删除列表中的某个item时，点击删除后若当前的适配器没有调用此方法（即没有mAdapter.notifyDataSetChanged()），IDE会报错。而这个方法也可以在Activity的onResume()方法中调用，从而在跳转过程后，回到当前页面仍然保持最新的列表状态。

    @Override
    public void notifyDataSetChanged() {
        super.notifyDataSetChanged();
    }

  整个MsgAdapter代码如下：

	
	/**
	  * Created by Carson on 2017/7/15.
	  */
		
	public class MsgAdapter extends ArrayAdapter<Msg>{
	
	    private int resourceId;
	    private List<Msg> message = new ArrayList<Msg>();
	
	    public MsgAdapter(@NonNull Context context, @LayoutRes int resource) {
	        super(context, resource);
	        resourceId = resource;
	    }
	
	    public MsgAdapter(@NonNull Context context, @LayoutRes int resource, @NonNull List<Msg> objects) {
	        super(context, resource, objects);
	        resourceId = resource;
	        this.message = objects;
	    }
	
	    //创建item的临时存储器，提升上下拉动时的加载的速度
	    class ViewHolder {
	        ImageView head;
	        TextView name, msg;
	    }
	
	    //获取列表的个数
	
	    @Override
	    public int getCount() {
	        return message.size();
	    }
	
	    //获取item的位置
	
	    @Override
	    public int getPosition(@Nullable Msg item) {
	        return super.getPosition(item);
	    }
	
	    //获取当前位置的item
	
	    @Nullable
	    @Override
	    public Msg getItem(int position) {
	        return super.getItem(position);
	    }
	
	
	    //获取item的view
	
	    @NonNull
	    @Override
	    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {
	        View item;
	        ViewHolder mViewHolder;
	        //获取当前位置的position以填充布局
	        Msg msg = getItem(position);
	        //如果当前view为空，则将当前view填充入viewholder中，否则从viewholder中回调出来
	        if (convertView == null) {
	            //将当前view加载出来
	            item = LayoutInflater.from(getContext()).inflate(resourceId, null);
	            mViewHolder = new ViewHolder();
	            mViewHolder.head = (ImageView) item.findViewById(R.id.iv_head);
	            mViewHolder.name = (TextView) item.findViewById(R.id.tv_name);
	            mViewHolder.msg = (TextView) item.findViewById(R.id.tv_msg);
	            item.setTag(mViewHolder);
	        } else {
	            item = convertView;
	            mViewHolder = (ViewHolder) item.getTag();
	        }
	
	        mViewHolder.head.setImageResource(msg.getImageResource());//设定头像
	        mViewHolder.name.setText(msg.getName());//设定用户名
	        mViewHolder.msg.setText(msg.getMsg());//设定消息
	
	        return item;
	    }
	
	    @Override
	    public void notifyDataSetChanged() {
	        super.notifyDataSetChanged();
    	}
	}

Activity的代码：

    public class MsgActivity extends AppCompatActivity {

	    private List<Msg> mList;
	    private MsgAdapter mAdapter;
	    private ListView mListView;
	
	    //随机定义一些图片、用户名、内容
	    private int []head = {
	            R.drawable.recommend_head1,
	            R.drawable.recommend_head2,
	            R.drawable.recommend_head3,
	            R.drawable.recommend_head4
	    };
	
	    private String []name = {
	            "King",
	            "Lily",
	            "July",
	            "Kelvin"
	    };
	
	    private String []message = {
	            "I am message one.",
	            "I am message two.",
	            "I am message three.",
	            "I am message four."
	    };
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        getSupportActionBar().hide();
	        setContentView(R.layout.activity_msg);
	
	        initViews();
	
	    }
	
	    private void initViews() {
	        mList = new ArrayList<Msg>();
	        mAdapter = new MsgAdapter(this, R.layout.layout_msg, mList);
	        mListView = (ListView) findViewById(R.id.lv_msg);
	
	        mListView.setAdapter(mAdapter);
	
	        Random r = new Random();
	
	        //可以从这里初始化listView
	        for (int i = 1; i <= 10; i++){
	            Msg msg = new Msg(head[r.nextInt(4)], "" + name[r.nextInt(4)], "" + message[r.nextInt(4)]);
	            //将实例化后的对象加入到list中，然后list通过adapter适配于listview中
	            mList.add(msg);
	        }
	
	    }
	}




  