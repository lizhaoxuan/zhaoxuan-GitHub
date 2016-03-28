##P层-Activity生命周期的更优调用
寻找最简单的

========================


###从前
P层抽取了Activity（View）层的业务。所以原本Activity生命周期方法内的业务也要移到P中，我们一般通过这样的方式：

代码我们简化了很多：


	class Presenter {

		....

		public Presenter(View view){
			this.view = view;
		}

    	protected void onStart() {
       		//dosoming
    	}

    	protected void onResume() {
			//dosoming
    	}
	}

	class MainActivity  extends AppCompatActivity {

		private Presenter presenter;
		。。。

		@Override
    	protected void onStart() {
        	super.onStart();
        	presenter.onStart();
    	}

    	@Override
    	protected void onResume() {
        	super.onResume();
        	presenter.onResume();
    	}
		。。。
	}	
	

方法总结起来就是，需要的时候在activity的生命周期方法中调用P的方法。乍一看似乎没有问题。但将业务往大了想：你有100个activity，几乎每个activity的生命周期方法都涉及P。还这样写，**你不累么？**

这是非常明显的重复代码，而且一成不变，几乎所有的Presenter都需要。恩~那么这样的代码怎么优化呢？模板啊~这多像模板啊。

Activity本身的生命周期方法就是使用了模板设计模式，我们要做的，就是想办法设计一个双模板（Activity模板、Presenter模板），并且让这两个模板结合起来。


###设计

模板模式很简单，但是我们这里并不是说简单建立两个BaseActivity和BasePresenter就行了。有这样一个问题：

#####问题：

- 不能使用单纯的父类或接口。

	因为你不会愿意每次调用都需要强转的，比如这样
	
		((LoginPresenter)presenter).login();


#####解决：

1.避免类型强制转换。


说到避免强制转换，你第一个应该想到的就是泛型。所以我们的解决方案就是下面这样：

	public class BaseActivity<T extends BasePresenter> extends AppCompatActivity {

    	private T presenter;

    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
    	}

    	public T setPresenter(Class<T> pClazz) {
        	try {
            	presenter = pClazz.newInstance();
        	} catch (Exception e) {
            	e.printStackTrace();
        	}
        	return presenter;
    	}

    	final public T getPresenter() {
        	return presenter;
    	}
    	
    	@Override
    	protected void onStart() {
        	super.onStart();
        	presenter.onStart();
    	}

    	@Override
    	protected void onResume() {
    	    super.onResume();
    	    presenter.onResume();
    	}
    	。。。
    	
    }


这里要注意三个点：

- 限制泛型类型：

		<T extends BasePresenter>
		
- 由子类决定初始化哪个Presenter

		public T setPresenter(Class<T> pClazz){}
		
- 无法复写的get方法
	
		final public T getPresenter() {
        	return presenter;
    	}
    	


子Activity的实现如下：

	public class MainActivity extends BaseActivity<MainPresenter>
        implements IMainView {

    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
        	setContentView(R.layout.activity_main);
			
			//设置要实现的Presenter
        	setPresenter(MainPresenter.class);
        	//不需要强转类型，编译时就已经决定是那种Presenter
        	getPresenter().getData();
		}
	
	。。。
	
	}
	

BasePresenter的实现如下：

	public class BasePresenter {

    	protected void onStart() {
    	}

    	protected void onResume() {
    	}

    	protected void onPause() {
    	}

    	protected void onStop() {
    	}

    	protected void onDestroy() {
    	}
	}


子Presenter的实现：

	public class MainPresenter extends BasePresenter implements IMainPresenter {

    	private IMainView mainView;

    	public void setMainView(IMainView mainView) {
        	this.mainView = mainView;
    	}

    	@Override
    	public void getData() {
        	mainView.showToast("asdasd");
    	}

    	@Override
    	protected void onDestroy() {
        	super.onDestroy();
        	Log.d("TAG","onDestroy");
    	}
    	
    	。。。	
    }

###结论

最后具体方便在哪了呢？

你完全可以把你的Presenter当做activity来用，需要复写什么，直接写就好了，不用再劳心劳力的在子activity当中调用。

这里我们只是实现了6个生命周期方法，你也可以添加其他Activity方法到模板中。任何activity方法。

**使用架构的初衷是使开发变得更简单。而不是华丽。**


