## RecyclerView的Adapter ##

RecylerView的出现，可以说是为实现更复杂的页面布局做了更加方便的代码操作。比如当你要实现横向滑动的列表或者瀑布流的视图，若用ListView和GridView实现，还是要花费一点时间的，而用RecyclerView实现这些效果还是相对容易的。



而RecyclerView的使用需要我们添加依赖：



	dependencies {

	    compile 'com.android.support:recyclerview-v7:25.0.1'

	}



接下来我们就可以在主活动的布局文件添加该控件：



	    <android.support.v7.widget.RecyclerView

	        android:id="@+id/rv"

	        android:layout_width="match_parent"

	        android:layout_height="match_parent">

	    </android.support.v7.widget.RecyclerView>



相信大家对ListView的使用也有了一定心得，而RecyclerView的使用与前者也大同小异。我们可以自定义一个Adapter来适配该控件。



当我们继承了RecyclerView.Adapter< VH >后，要求重写的方法有三个：



- int getItemCount()

- RecyclerView.ViewHolder onCreateViewHolder(ViewGroup parent, int viewType)

- void onBindViewHolder(RecyclerView.ViewHolder holder, int position)



第一个就不讲了，也就是返回的是RecylerView的item的数目。



而在讲第二第三个方法之前，我们先来看看这个尖括号里的< VH >，从RecyclerView.Adapter的抽象类来看，VH是继承自RecyclerView.ViewHolder：



	public static abstract class Adapter<VH extends ViewHolder>{

		...

	}



而这个ViewHolder跟ListView里的也是大同小异，为的是为我们保存RecyclerView中的item视图，以便复用及提升控件性能。因此我们类比ListView，在Adapter类里写一个内部类继承自RecyclerView.ViewHolder。



    class MyViewHolder extends RecyclerView.ViewHolder {



        TextView tv;



        public MyViewHolder(View itemView) {

            super(itemView);

            tv = (TextView) itemView.findViewById(R.id.tv_itemName);

        }

    }



将MyViewHolder写进Adapter< MyViewHolder >内，则上述第二第三个方法也对应地变成





- MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType)

- void onBindViewHolder(MyViewHolder holder, int position)



onCreateViewHolder()方法：



viewType指的是item的类型，即自己定义，如headerView、footerView等若加载不同的布局，则需要根据个人喜好自定义viewType。而此方法其实就是对自定义的holder实例化，为之加载布局。



	@Override

    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {



        MyViewHolder holder = 

				new MyViewHolder(LayoutInflater.from(parent.getContext())

				.inflate(R.layout.layout_msg, parent, false));

		return holder;

		}



onBindViewHolder()方法：



这个方法是主要是绑定数据给ViewHolder。如TextView的setText方法、ImageView的setImageResource方法都可在此方法中写。



    @Override

    public void onBindViewHolder(MyViewHolder holder, int position) {

        holder.tv.setText(items.get(position));

    }



那么接下来就在活动里写代码了，就直接上代码吧：



	public class RVActivity extends AppCompatActivity {



	    private RecyclerView mRecyclerView;

	    private MyAdapter myAdapter;

	

	    @Override

	    protected void onCreate(Bundle savedInstanceState) {

	        super.onCreate(savedInstanceState);

	        setContentView(R.layout.layout_flow);

	

	        mRecyclerView = (RecyclerView) findViewById(R.id.rv);

	

	        myAdapter = new MyAdapter();

	

	        initData();

	

			//线性布局展示

			//mRecyclerView.setLayoutManager(new LinearLayoutManager(this));

			//网格布局展示，第二个参数指展示的列数

	        //mRecyclerView.setLayoutManager(new GridLayoutManager(this, 4));

			//瀑布流布局展示，第一个参数为展示的行数或者列数，视第二个参数而定。

	        mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(2, 

			StaggeredGridLayoutManager.VERTICAL));

	        mRecyclerView.setAdapter(myAdapter);

	

	    }

	

	    private void initData() {

	        for(int i = 'a'; i <= 'z'; i++){

	            myAdapter.getItems().add("" + (char) i);

	        }

	    }



	}



至于分隔线的问题，由于RecyclerView并没有divider属性，所以如果只是要设置有空隙的话，在item的布局文件中将控件设置margin就好；若是分隔线有自定义样式，可以自定义分隔线类继承自ItemDecoration，重写该类的方法来实现自己的自定义分隔线。



而在item的点击事件的实现中，RecyclerView内部是没有类似ListView中的setOnItemClickListener方法，从而使得我们需要在其适配器中进一步去设置点击事件。而这样子与ListView相比，你也许会觉得很麻烦，其实也就这一点稍微需要花一点点功夫而已，但是它所能实现的功能所需要写的代码可是要比前者少很多。毕竟在ListView中整个布局是作为一个item整体来点击的，而RecyclerView则可以将布局内的每一控件都自由设置点击事件。由此看来，的确降低了代码的耦合性，故抛弃item的设置点击方法而由开发者自由设置是有理由的。



说了一堆，真正的实现，其实我们只需要在onCreateViewHolder的方法里为我们的控件设置点击事件即可。



	@Override

    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {



        final MyViewHolder holder = new MyViewHolder(LayoutInflater

						.from(parent.getContext())

						.inflate(R.layout.layout_msg, parent, false));

		

		holder.tv.setOnClickListener(new View.OnClickListener(){

			@Override

            public void onClick(final View view) {

				int position = holder.getAdapterPosition();

				Toast.makeText(view.getContext(), 

							"这是第" + item.get(position) + "位", 

							Toast.LENGTH_SHORT).show();

			}

		});





	}



这样子便完成了对item中的TextView控件的点击事件。大家可以以此类推为布局中的控件设置各类不同的点击事件（包括setOnLongClickListener），从而实现各类想要的功能效果。





同时，我又根据之前一篇ListView博文中的消息类用RecyclerView又写了一下。下面贴一下代码，也请各位指出一下不足之处。



**MyAdapter适配器类:**



	public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder> {





	    private List<Msg> items;

	

	    public List<Msg> getItems() {

	        return items;

	    }

	

	    public MyAdapter(List<Msg> list) {

	        this.items = list;

	    }

	

	    @Override

	    public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {

	

	        final MyViewHolder holder = new MyViewHolder(LayoutInflater.from(parent.getContext()).inflate(R.layout.layout_msg, parent, false));

	

	        holder.iv.setOnClickListener(new View.OnClickListener() {

	            @Override

	            public void onClick(final View view) {

	                int position = holder.getAdapterPosition();

	                Msg msg = items.get(position);

	                final View itemView = LayoutInflater.from(view.getContext()).inflate(R.layout.rv_item, null, false);

	                AlertDialog dialog = new AlertDialog.Builder(view.getContext())

	                        .setView(R.layout.rv_item)

	                        .setIcon(msg.getImageResource())

	                        .setTitle(msg.getName())

	                        .setPositiveButton("登陆", new DialogInterface.OnClickListener() {

	                            @Override

	                            public void onClick(DialogInterface dialogInterface, int position) {

	                                EditText name = (EditText) itemView.findViewById(R.id.et_username);

	                                EditText pwd = (EditText) itemView.findViewById(R.id.et_pwd);

	                                Toast.makeText(view.getContext(), msg.getName() + "登陆成功", Toast.LENGTH_SHORT).show();

	                            }

	                        })

	                        .setNegativeButton("取消", new DialogInterface.OnClickListener() {

	                            @Override

	                            public void onClick(DialogInterface dialogInterface, int i) {

	                                dialogInterface.dismiss();

	                            }

	                        })

	                        .show();

	

	            }

	        });

	

	        return holder;

	    }

	

	    @Override

	    public void onBindViewHolder(MyViewHolder holder, int position) {

	        Msg msg = items.get(position);

	        holder.tv.setText(msg.getName());

	        holder.iv.setImageResource(msg.getImageResource());

	        holder.msg.setText(msg.getMsg());

	    }

	

	    @Override

	    public int getItemCount() {

	        return items.size();

	    }

	

	

	    class MyViewHolder extends RecyclerView.ViewHolder {

	

	        TextView tv;

	        ImageView iv;

	        TextView msg;

	

	        public MyViewHolder(View itemView) {

	            super(itemView);

	            tv = (TextView) itemView.findViewById(R.id.tv_name);

	            iv = (ImageView) itemView.findViewById(R.id.iv_head);

	            msg = (TextView) itemView.findViewById(R.id.tv_msg);

	        }

	    }

	}



**启动的活动类：**

	

	public class RVActivity extends AppCompatActivity {

	

	    private RecyclerView mRecyclerView;

	    private MyAdapter myAdapter;

	    private List<Msg> messages = new ArrayList<>();

	

	    private int []head = {

	            R.drawable.recommend_head1,

	            R.drawable.recommend_head2,

	            R.drawable.recommend_head3,

	            R.drawable.recommend_head4

	    };

	

	    private String []message = {

	            "I am message one.",

	            "I am message two.",

	            "I am message three.",

	            "I am message four."

	    };

	

	    private String []name = {

	            "King",

	            "Lily",

	            "July",

	            "Kelvin"

	    };

	

	    @Override

	    protected void onCreate(Bundle savedInstanceState) {

	        super.onCreate(savedInstanceState);

	        getSupportActionBar().hide();

	        setContentView(R.layout.layout_flow);

	

	        mRecyclerView = (RecyclerView) findViewById(R.id.rv);

	

	        myAdapter = new MyAdapter(messages);

	

	        initData();

	

	        //mRecyclerView.setLayoutManager(new LinearLayoutManager(this));

	

	        //mRecyclerView.setLayoutManager(new GridLayoutManager(this, 4));

	        mRecyclerView.setLayoutManager(new StaggeredGridLayoutManager(2, StaggeredGridLayoutManager.VERTICAL));

	        mRecyclerView.setAdapter(myAdapter);

	

	    }

	

	    private void initData() {

	        Random r = new Random();

	        StringBuilder builder = new StringBuilder();

			//这里只是为了突显瀑布流的视图效果

	        int length = r.nextInt(10) + 2;

	        String []newName = new String[length];

	        for (int j = 0; j < length; j++){

	            builder.append(name[r.nextInt(4)]);

	            newName[j] = builder.toString();

	        }

	

	        for(int i = 0; i <= 30; i++){

	            Msg msg = new Msg(head[r.nextInt(4)], newName[r.nextInt(length)], message[r.nextInt(4)]);

	            messages.add(msg);

	        }

	    }

	}





效果图：







![这里写图片描述](http://img.blog.csdn.net/20170730213458819?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvQ2Fyc29uV29v/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



