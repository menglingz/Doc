### 一、adapter

#### 1.BrowseAdapter

将浏览记录数据呈现在view上,作用是将子项(xml.layout)与RecyclerView的一个布局适配。

适配器：Adapter的作用就是ListView界面与数据之间的桥梁，当列表里的每一项显示到页面时，都会调用Adapter的getView方法返回一个View。

```Android
public class BrowseAdapter extends RecyclerView.Adapter<BrowseAdapter.ViewHolder> 
```

自定义adapter:类BrowseAdapter继承于RecyclerView.Adapter，并将泛型指定为自定义的内部类ViewHolder；ViewHolder理解为视图控件持有类，是一个囊括本类对象里所有控件的容器。

```
 public ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
        mActivity = viewGroup.getContext();
        View view=LayoutInflater.from(mActivity).
        inflate(R.layout.item_rv_collect_list,viewGroup,false);
        return new ViewHolder(view);
    }
```

ViewHolder通常出现在适配器里，为的是listview滚动的时候快速设置值，而不必每次都重新创建很多对象，从而提升性能。作用就是减少不必要的findViewById, 然后将底下的控件引用存在ViewHolder里面, 再在View.setTag(holder)把它放在view 里面, 这样一来下次就可以直接取了。

这个item需要我们用inflate函数把item_rv_clothing_list动态的加载进main布局，并且返回了一个用来获取item里控件并且对其进行操作的View对象。

其中，LayoutInflater这个类的作用类似于findViewById()，不同点LayoutInflater是用来找layout下xml布局文件，并且实例化！而findViewById()是找具体xml下的具体widget控件(如:Button,TextView等)。

```
viewHolder.itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (mItemListener!=null){
                        mItemListener.ItemClick(browse);
                    }
                }
            });
```

onClicklistener是一个接口,不能实例化,这就是一个匿名内部类,这个类实现onClickListener,然后被new 了,无形中传了一个对象进去,这个对象给了button/TextView中的mOnClicklistener,就是这家伙调用了onClick方法

#### 2.ClothingAdapter

将衣服数据呈现在view上

```
Glide.with(mActivity)
        .asBitmap()
        .load(clothing.getImg())
        .error(R.drawable.ic_error)
        .skipMemoryCache(true)
        .diskCacheStrategy(DiskCacheStrategy.NONE)
        .into(viewHolder.ivPhoto);
```

使用glide框架加载图片，Glide框架是google推荐的Android图片加载框架。Glide.with()一共有六个同名方法，分别为Glide.with(Context context),Glide.with(Activityactivity),Glide.with(FragmentActivity fragmentActivity),Glide.with(Fragmentfragment),Glide.with(android.app.Fragmenfragment),Glide.with(View view)。

可以分成四类，Glide.with(Activity)，Glide.with(Fragment), Glide.with(View),Glide.with(Context)。

```
 AlertDialog.Builder dialog = new AlertDialog.Builder(mActivity);
                        dialog.setMessage("确认要删除该衣服吗");
                        dialog.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                list.remove(clothing);
                                clothing.delete();
                                notifyDataSetChanged();
                                Toast.makeText(mActivity,"删除成功", Toast.LENGTH_LONG).show();
                                if (list!=null && list.size() > 0){
                                    rvFlowersList.setVisibility(View.VISIBLE);
                                    llEmpty.setVisibility(View.GONE);
                                }else {
                                    rvFlowersList.setVisibility(View.GONE);
                                    llEmpty.setVisibility(View.VISIBLE);
                                }
                            }
                        });
                        dialog.setNeutralButton("取消", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                dialog.dismiss();
                            }
                        });
                        dialog.show();
                        return false;
```

AlertDialog的使用很普遍，在应用中当你想要用户做出“是”或“否”或者其它各式各样的选择时，为了保持在同样的Activity和不改变用户屏幕，就可以使用AlertDialog。

AlertDialog的构找函数是protected，那就不能直接通过构找函数去构建，必须通过AlertDialog.Builder。AlertDialog是Dialog的一个直接子类，AlertDialog也是Android系统当中最常用的对话框之一。 

一个AlertDialog可以有两个以上的Button，可以对一个AlertDialog设置相应的信息。比如title，massage，setSingleChoiceItems，setPositiveButton，setNegativeButton等等.但不能直接通过AlertDialog的构造函数来生产一个AlertDialog。只能通过Builder去构造。

简单来说：AlertDialog.Builder是AlertDialog封装的一个内部类，实现了构造器模式，所有AlertDialog需要设置一些属性必须通过构造器去构造。Builder设计模式可以很好地控制参数的个数以及灵活的组合多种参数。

```
    public interface ItemListener{
        void ItemClick(Clothing Flowers);
    }
```

声明一个借口，当当用户选择或取消选择项目时调用。

#### 3.OrderAdapter

将订单数据呈现在view上

```
User user = DataSupport.where("account = ? ", order.getAccount()).findFirst(User.class);
```

DataSupport用于创建模拟数据与进行数据检查的类

#### 4.UserAdapter

将用户数据呈现在view上

### 二、activity

#### 1.AddClothingActivity

添加衣服数据，编辑衣服详细信息

 mActionBar.setData(myActivity,"编辑衣服信息", R.drawable.ic_back, 0, 0, getResources().getColor(R.color.colorPrimary), new ActionBar.ActionBarClickListener() {
            @Override
            public void onLeftClick() {
                finish();
            }

​            @Override
​            public void onRightClick() {
​            }
​        });

调用ActionBar自定义标题栏

#### 2.BrowseActivity

浏览记录功能实现

#### 3.ClothingDetailActivity

查看衣服详细信息

```
setContentView(R.layout.activity_clothing_detail);
```

在Acitivty中setContentView()用来设置布局文件，将布局文件添加进窗口。setContentView方法是在MainActiviy中显示activity_main中所定义的内容,将控件添加进来。

```
final Clothing clothing = (Clothing) getIntent().getSerializableExtra("clothing");
```

Activity之间通过Intent传递值，获取clothing实体中的数据

例：

```
public class CustomeClass implements Serializable{        
    private static final long serialVersionUID = -7060210544600464481L;   
    private String name;   
    private String id;   
    private int age;   
    private String sex;   
       
    public String getName() {   
        return name;   
     }   
    public void setName(String name) {   
        this.name = name;   
     }   
    public String getId() {   
        return id;   
     }   
    public void setId(String id) {   
        this.id = id;   
     }   
    public int getAge() {   
        return age;   
     }   
    public void setAge(int age) {   
        this.age = age;   
     }   
    public String getSex() {   
        return sex;   
     }   
    public void setSex(String sex) {   
        this.sex = sex;   
     }   
  
}  
发送部分

CustomeClass cc = new CustomeClass();   
cc.setAge(21);   
cc.setId("123456");   
cc.setName("mingkg21");   
cc.setSex("男");   
  
Intent intent = new Intent(this, PersonInfo.class);   
intent.putExtra("PERSON_INFO", cc);   
startActivity(intent);  
接收部分

Intent intent = getIntent();   
CustomeClass cc = CustomeClass)intent.getSerializableExtra("PERSON_INFO");   
setTextView(R.id.id, cc.getId());   
setTextView(R.id.name, cc.getName());   
setTextView(R.id.sex, cc.getSex());   
setTextView(R.id.age, String.valueOf(cc.getAge())); 
```

在实体类中创建数据对象，在适配器中将相应的数据对象发送，在activty中接收。

```
intent.putExtra("user",user)
```

通过此标识发送的信息，并在activity中通过name接收相应的信息。

#### 4.LoginActivity

登录界面功能

```
        //选择类型
        rgType.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup group, int checkedId) {
                SPUtils.put(activity,SPUtils.IS_ADMIN,checkedId == R.id.rb_admin);
            }
        });
```

保存注册的数据，调用自定义方法Sputils中的put方法。

```
InputMethodManager inputMethodManager= (InputMethodManager) v.getContext().getSystemService(Context.INPUT_METHOD_SERVICE);
                inputMethodManager.hideSoftInputFromWindow(v.getWindowToken(),0);
```

InputMethodManager是整个输入法框架（IMF）结构的核心API，应用程序之间进行调度和当前输入法交互。

```
SPUtils.put(LoginActivity.this,"account",account);
Intent intent = new Intent(activity, MainActivity.class);
```

登录成功后保存数据，然后跳转到主页面。

#### 5.MainActivity

```
super.onCreate(savedInstanceState);
```

onCreate函数用来初始化Activity实例对象，在前面Activity博客中我们知道onCreate的方法是在Activity创建时被系统调用，是一个Activity生命周期的开始.

```
 private void setViewListener() {
        rbFlowers.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mActionBar.setTitle("衣服商城");
                switchFragment(0);
            }
        });
        rbFlowersData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if ("".equals(mAccount)) {////未登录,跳转到登录页面
                    MyApplication.Instance.getMainActivity().finish();
                    startActivity(new Intent(myActivity, LoginActivity.class));
                }else {//已经登录
                    mActionBar.setTitle("订单");
                    switchFragment(1);
                }
            }
        });
        rbUserManage.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                mActionBar.setTitle("用户管理");
                switchFragment(2);
            }
        });
        rbUser.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if ("".equals(mAccount)) {////未登录,跳转到登录页面
                    MyApplication.Instance.getMainActivity().finish();
                    startActivity(new Intent(myActivity, LoginActivity.class));
                }else {//已经登录
                    mActionBar.setTitle("我的");
                    switchFragment(mIsAdmin?3:2);
                }
            }
        });
    }
```

事件监听器：setViewListener()，当用户点击不同的界面切换不同的fragment。

```
private void initView() {
	mIsAdmin = (Boolean) SPUtils.get(myActivity,SPUtils.IS_ADMIN,false);
	mAccount = (String) SPUtils.get(myActivity,SPUtils.ACCOUNT,"");
    //设置导航栏图标样式
    Drawable 	iconNews=getResources().getDrawable(R.drawable.selector_main_rb_home);//设置主页图标样式
    iconNews.setBounds(0,0,68,68);//设置图标边距 大小
    rbFlowers.setCompoundDrawables(null,iconNews,null,null);//设置图标位置
    rbFlowers.setCompoundDrawablePadding(5);//设置文字与图片的间距
    Drawable iconNewsData=getResources().getDrawable(R.drawable.selector_main_rb_buy);//设置主页图标样式
    iconNewsData.setBounds(0,0,68,68);//设置图标边距 大小
    rbFlowersData.setCompoundDrawables(null,iconNewsData,null,null);//设置图标位置
    rbFlowersData.setCompoundDrawablePadding(5);//设置文字与图片的间距
    Drawable iconSetting=getResources().getDrawable(R.drawable.selector_main_rb_manage);//设置主页图标样式
    iconSetting.setBounds(0,0,60,60);//设置图标边距 大小
    rbUserManage.setCompoundDrawables(null,iconSetting,null,null);//设置图标位置
    rbUserManage.setCompoundDrawablePadding(5);//设置文字与图片的间距
    Drawable iconUser=getResources().getDrawable(R.drawable.selector_main_rb_user);//设置主页图标样式
    iconUser.setBounds(0,0,55,55);//设置图标边距 大小
    rbUser.setCompoundDrawables(null,iconUser,null,null);//设置图标位置
    rbUser.setCompoundDrawablePadding(5);//设置文字与图片的间距
    rbUserManage.setVisibility(mIsAdmin? View.VISIBLE: View.GONE);
    switchFragment(0);
    rbFlowers.setChecked(true);
}
```

initView()方法：初始化界面，当用户点击新导航栏图标时，更换样式。

```
private void switchFragment(int fragmentIndex) {
        //在Activity中显示Fragment
        //1、获取Fragment管理器 FragmentManager
        FragmentManager fragmentManager = this.getFragmentManager();
        //2、开启fragment事务
        FragmentTransaction transaction = fragmentManager.beginTransaction();

        //懒加载 - 如果需要显示的Fragment为null，就new。并添加到Fragment事务中
        if (fragments[fragmentIndex] == null) {
            if (mIsAdmin){
                switch (fragmentIndex) {
                    case 0://NewsFragment
                        fragments[fragmentIndex] = new ClothingFragment();
                        break;
                    case 1://CollectFragment
                        fragments[fragmentIndex] = new OrderFragment();
                        break;
                    case 2://UserManageFragment
                        fragments[fragmentIndex] = new UserManageFragment();
                        break;
                    case 3://UserFragment
                        fragments[fragmentIndex] = new UserFragment();
                        break;
                }
            }else {
                switch (fragmentIndex) {
                    case 0://NewsFragment
                        fragments[fragmentIndex] = new ClothingFragment();
                        break;
                    case 1://CollectFragment
                        fragments[fragmentIndex] = new OrderFragment();
                        break;
                    case 2://UserFragment
                        fragments[fragmentIndex] = new UserFragment();
                        break;
                }
            }

            //==添加Fragment对象到Fragment事务中
            //参数：显示Fragment的容器的ID，Fragment对象
            transaction.add(R.id.ll_main_content, fragments[fragmentIndex]);
        }
```

switchFragment()方法:切换fragment,设置fragment的索引，并将其存入对象中。

Fragment是Android3.0以后引入的新的api，为了适配大屏的平板。在普通手机开发的过程中，使用Fragment能实现一个界面的多次使用，能加快效率。Fragment可以被认为是Activity界面的一个布局，其依赖于Activity，但是拥有自己的活动事件与生命周期。可以通过替换Activity中的Fragment实现界面的优化处理。当进入主页面后，点击任何（用户，订单，我的），这个页面都可以充斥整个屏幕。

#### 6.OpeningActivity

衣服商城打开界面

```
public class OpeningActivity extends AppCompatActivity {}
```

继承AppCompatActivity，界面最上面会出现一个ActionBar，默认显示项目的名称Toolbar(普通activity没有上面标题栏)

AppcompaActivity相对于Activity的主要的两点变化；
1：主界面带有toolbar的标题栏；
2，theme主题只能用android:theme=”@style/AppTheme （appTheme主题或者其子类），而不能用android:style，否则会提示错误。

```
setContentView(R.layout.activity_opening);
try {
	initView();
} catch (IOException | JSONException e) {
	e.printStackTrace();
}
```

e是Throwable的实例异常对象，用在catch语句中，相当于一个形参，一旦try捕获到了异常，那么就将这个异常信息交给e，由e处理，printStackTrace()是异常类的一个方法。printStackTrace()方法的意思是：在命令行打印异常信息在程序中出错的位置及原因。

```
public void run() {
	if((getIntent().getFlags() & Intent.FLAG_ACTIVITY_BROUGHT_TO_FRONT) != 0)
	{
    	finish();
        return;
    }
```

Activity的`Intent Flags`,它是Activity的标记位，常用于Activity的场景中，与Activity的启动模式有着密切的联系

FLAG_ACTIVITY_BROUGHT_TO_FRONT：如果activity在task存在，拿到最顶端,不会启动新Activity。

#### 7.PasswordActivity

修改密码功能界面

#### 8.PersonActivity

修改个人信息功能界面

#### 9.RegisterActivity

注册功能界面

#### 10.UserDetailActivity

编辑用户个人信息功能界面

```
mUser = (User) getIntent().getSerializableExtra("user");
```

接收用户信息，并将其传入一个对象，然后获取这个对象（用户）中的用户详细数据。

#### 11.ClothingFragment

衣服子界面

```
public class ClothingFragment extends Fragment {}
```

继承fragment

Fragment是Android3.0以后引入的新的api，为了适配大屏的平板。在普通手机开发的过程中，使用Fragment能实现一个界面的多次使用，能加快效率。Fragment可以被认为是Activity界面的一个布局，其依赖于Activity，但是拥有自己的活动事件与生命周期。可以通过替换Activity中的Fragment实现界面的优化处理。当进入主页面后，点击任何（用户，订单，我的），这个页面都可以充斥整个屏幕。

```
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle 											savedInstanceState) {
        View view=inflater.inflate(R.layout.fragment_flowers,container,false);
```

inflate()的作用就是将一个用xml定义的布局文件查找出来，注意与findViewById()的区别，inflate是加载一个布局文件，而findViewById则是从布局文件中查找一个控件。

关于LayoutInflater类inflate(int resource, ViewGroup root, boolean attachToRoot)方法三个参数的含义：

resource：需要加载布局文件的id，意思是需要将这个布局文件中加载到Activity中来操作。

root：需要附加到resource资源文件的根控件，什么意思呢，就是inflate()会返回一个View对象，如果第三个参数attachToRoot为true，就将这个root作为根对象返回，否则仅仅将这个root对象的LayoutParams属性附加到resource对象的根布局对象上，也就是布局文件resource的最外层的View上，比如是一个LinearLayout或者其它的Layout对象。

attachToRoot：是否将root附加到布局文件的根视图上

```
tabTitle.setTabMode(TabLayout.MODE_SCROLLABLE);

<com.google.android.material.tabs.TabLayout
            android:id="@+id/tab_title"
            android:layout_width="match_parent"
            android:layout_height="40dp"
            app:tabMode="fixed"
            app:tabGravity="fill"
            android:background="@color/colorWhite"
            app:tabIndicatorColor="@color/colorPrimary"
            app:tabTextColor="@color/colorBlack"
            app:tabSelectedTextColor="@color/colorPrimary"/>
```

TabLayout是Android support中的一个控件，Google在升级了AndroidX之后，将TabLayout迁移到material包下面去了com.google.android.material.tabs.TabLayoutTabLayout一般结合ViewPager+Fragment的使用实现滑动的标签选择器。比如衣服的样式滑动选择。

```
 GridLayoutManager layoutManager = new GridLayoutManager(myActivity, 2);//两列
 layoutManager.setOrientation(LinearLayoutManager.VERTICAL);
```

水平分页布局管理器，将衣服分成两列展示

```
public void ItemClick(Clothing clothing) {boolean isAdmin = (boolean) 			     							SPUtils.get(myActivity,SPUtils.IS_ADMIN,false);
	String account = (String) SPUtils.get(myActivity,SPUtils.ACCOUNT,"");
    if ("".equals(account)) {//未登录,跳转到登录页面
                    MyApplication.Instance.getMainActivity().finish();
                    startActivity(new Intent(myActivity, LoginActivity.class));
    }else {//已经登录
    	Intent intent;
        if (isAdmin){
        	intent = new Intent(myActivity, AddClothingActivity.class);
        }else {
        	intent = new Intent(myActivity, ClothingDetailActivity.class);
                    }
        intent.putExtra("clothing",clothing);
        startActivityForResult(intent,100);
        	}
       }
});
```

```
public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 100 && resultCode == RESULT_OK){
            loadData();//加载数据
        }
    }
```

在一个主界面(主Activity)通过意图跳转至多个不同子Activity上去，当子模块的代码执行完毕后再次返回主页面，将子activity中得到的数据显示在主界面/完成的数据交给主Activity处理。这种带数据的意图跳转需要使用activity的onActivityResult()方法。

**（1）startActivityForResult([Intent](http://blog.csdn.net/sjf0115/article/details/7385152) intent, int requestCode);**

　　 第一个参数：一个Intent对象，用于携带将跳转至下一个界面中使用的数据，使用putExtra(A,B)方法，此处存储的数据类型特别多，基本类型全部支持。

　　 第二个参数：如果> = 0,当Activity结束时requestCode将归还在onActivityResult()中。以便确定返回的数据是从哪个Activity中返回，用来标识目标activity。

　　与下面的resultCode功能一致，感觉Android就是为了保证数据的严格一致性特地设置了两把锁，来保证数据的发送，目的地的严格一致性。

**（2）onActivityResult(int requestCode, int resultCode, [Intent](http://blog.csdn.net/sjf0115/article/details/7385152) data)**

　　第一个参数：这个整数requestCode用于与startActivityForResult中的requestCode中值进行比较判断，是以便确认返回的数据是从哪个Activity返回的。

　　第二个参数：这整数resultCode是由子Activity通过其setResult()方法返回。适用于多个activity都返回数据时，来标识到底是哪一个activity返回的值。

　　第三个参数：一个Intent对象，带有返回的数据。可以通过data.getXxxExtra( );方法来获取指定数据类型的数据，

#### 12.OrderFragment

订单子界面功能

#### 13.UserFragment

我的个人页面功能

#### 14.UserManagerFragment

用户管理功能呢页面

### 三、util

#### 1.RecyclerViewSpaces

控制item（衣服）之间的间距

RecyclerView是**support-v7包**中的**新组件**，是一个强大的滑动组件，与经典的ListView相比，同样拥有item回收复用的功能，这一点从它的名字Recyclerview即回收view也可以看出.

提供了一种**插拔式的体验**，**高度的解耦**，异常的灵活，针对一个Item的显示RecyclerView专门抽取出了**相应的类**，来**控制Item的显示**，使其的扩展性非常强。

设置**布局管理器**以控制**Item**的**布局方式**，**横向**、**竖向**以及**瀑布流**方式
 例如：你想控制横向或者纵向滑动列表效果可以通过**LinearLayoutManager**这个类来进行控制(与GridView效果对应的是**GridLayoutManager**,与**瀑布流**对应的还**StaggeredGridLayoutManager**等)。也就是说RecyclerView不再拘泥于ListView的线性展示方式，它也可以实现GridView的效果等多种效果。

可设置**Item的间隔样式**（可绘制）
 通过**继承RecyclerView的ItemDecoration**这个类，然后针对自己的业务需求去书写代码。

可以控制**Item增删的动画**，可以通过**ItemAnimator**这个类进行控制，当然针对增删的动画，RecyclerView有其自己默认的实现。

#### 2.Sputils

数据持久化工具类

```
public static void put(Context context, String key, Object object) {
```

保存数据的方法

```
public static Object get(Context context, String key, Object defaultObject) {
```

获取数据的方法

```
public static void remove(Context context, String key) {
```

删除数据中的值

### 四、xml

#### 1.RelativeLayout

相对布局可以让子控件相对于兄弟控件或父控件进行布局，可以设置子控件相对于兄弟控件或父控件进行上下左右对齐。RelativeLayout能替换一些嵌套视图，当我们用LinearLayout来实现一个简单的布局但又使用了过多的嵌套时，就可以考虑使用RelativeLayout重新布局。相对布局就是一定要加Id才能管理。

#### 2.ScrollView

ScrollView称为滚动视图，滚动条控件是当在一个屏幕的像素显示不下的时候，可以采用滑动的方式，显示在UI上

垂直滚动：ScrollView
水平滚动：HorizontalScrollView 

ScrollView的子元素只能有一个，所以得增加一个LinearLayout布局，把其他按键放在这个LinearLayout中，那么ScrollViewd的子元素就只有一个LinearLayout了，而LinearLayout的子元素不限制。

#### 3.TextView

TextView是用来显示文本的组件，文本显示控件

#### 4.EditText

EditText:文本框控件

#### 5.Spinner

下拉列表组件，Spinner提供了从一个数据集合中快速选择一项值的办法。默认情况下Spinner显示的是当前选择的值，点击Spinner会弹出一个包含所有可选值的dropdown菜单，从该菜单中可以为Spinner选择一个新值。

下拉列表的展示方式有两种，一种是在当前下拉框的正下方展示列表，此时把spinnerMode属性设置为dropdown;另一种是在页面中部以对话框形式展示列表，此时把SpinnerMode属性设置为dialog

#### 6.ImageView

图像视图，直接继承自View类，它的主要功能是用于显示图片，实际上它不仅仅可以用来显示图片，任何Drawable对象都可以使用ImageView来显示。ImageView可以适用于任何布局中，并且Android为其提供了缩放和着色的一些操作。

#### 7.RadioGroup

RadioGroup是可以容纳多个RadioButton的容器，单选框组件

RadioButton即单选框，是一种基础的UI控件。RadioGroup为我们提供了RadioButton单选按钮的容器，RadioButton通常放于RadioGroup容器中进行使用。RadioButton的选中状态，在xml文件中可以使用android:checked=""来进行设置，选中就设置为true，没选中就设置为false。

#### 8.TabLayout

TabLayout是Android support中的一个控件，Google在升级了AndroidX之后，将TabLayout迁移到material包下面去了com.google.android.material.tabs.TabLayoutTabLayout一般结合ViewPager+Fragment的使用实现滑动的标签选择器。比如衣服的样式滑动选择。

#### 9.FloatingActionButton

`FloatingActionButton`是5.0版本出现的控件，显示一个圆形悬浮按钮。需要添加`Design`依赖库，并且使用`Theme.AppCompat`主题

#### 10.RecyclerView

RecyclerView它可以说是一个增强版的ListView。

相对于 ListView、GridView 这类滑动列表组件来说，RecyclerView 具有：

（1）使用更加灵活
通过设置布局管理器，可轻松实现组件列表垂直滑动或水平滑动效果，给列表的 item 项设置【增加或删除】动画，当列表组件滑动到顶端或底部时，带有波纹效果，更显列表立体感
（2）代码高度解耦
代码高幅度降低了耦合度，维护起来非常方便
（3）拓展方便
轻松实现 ListView 及 Gridview 组件的相关列表效果，学会 RecyclerView，相当于学会了 ListView + Gridview，难得一见的【瀑布流】列表滑动效果。

#### 11.ListView

ListView是比较常用的组件，它以列表的形式展示具体内容，并且能够根据数据的长度自适应显示

### 五、剩余

#### 1.ActionBar

自定义标题栏view

#### 2.MyApplication

声明全局方法

#### 3.数据库

使用litepal数据库，LitePal 是一款开源的 Android 数据库框架，它采用了对象关系映射（ORM）的模式，将我们平时使用的一些数据库（比如 Sqlite）功能进行了封装。

使用LitePal添加数据,只需要创建出模型类的实例,然后将所有的要存储的数据设置好,最后调用一下save()方法就可以了,LitePal要求所有的实体类都要继承自DataSupport这个类,模型类必须继承DataSupport类

litepal.xml:用于设定数据库的名字，用于设定数据库的版本号，用于设定所有的映射模型
