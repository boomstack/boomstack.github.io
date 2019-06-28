---

layout: post 

title: "Fragment回退栈与事务" 

date: 2017-03-05 22:12:37 +0800 

categories: Android

---

一句话总结，回退栈管理的是事务（Transaction），栈里的数据结构是事务，不是fragment本身。
```java
FragmentManager manager = getSupportFragmentManager(); //（这是v4包的，app也有相关方法）
FragmentTransaction transaction = manager.beginTransaction();
```
FragmentManager和FragmentTransaction是最核心的两个类，一个是管理者，一个是被管理者。FragmentManager里边维护了回退栈，如下：
```java
ArrayList<BackStackRecord> mBackStack;
```
BackStackRecord是FragmentTransaction的实现类，核心的数据结构是Op类，记录了fragment的一些状态。
```java
    static final class Op {
        Op next;
        Op prev;
        int cmd;
        Fragment fragment;
        int enterAnim;
        int exitAnim;
        int popEnterAnim;
        int popExitAnim;
        ArrayList<Fragment> removed;
    }
```
要实现在Activity里管理fragment需要继承FragmentActivity。
## DEMO

其中一个fragment长这样，其他类似，最好不要在fragment里处理具体的点击逻辑和fragment替换等操作，要放到宿主Activity里控制。
```java
public class FragmentOne extends Fragment {
    public static final String TAG = "fragment_one";
    private Button btnOne;
    private FragmentOneBtnClickListener listener;

    interface FragmentOneBtnClickListener {
        void onFragmentOneBtnClick();
    }

    public void setOnFragmentBtnClickListener(FragmentOneBtnClickListener listener) {
        this.listener = listener;
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_one, container,false);
        btnOne = (Button) view.findViewById(R.id.btn_frag_one);
        btnOne.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (listener != null) {
                    listener.onFragmentOneBtnClick();
                }
            }
        });
        return view;
    }
}

```
这是宿主Activity
```java
public class SecondActivity extends AppCompatActivity implements FragmentOne.FragmentOneBtnClickListener,
        FragmentTwo.OnTwoClickListener, FragmentThree.OnThreeClickListener {
    private FragmentOne fragmentOne;
    private FragmentTwo fragmentTwo;
    private FragmentThree fragmentThree;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_second);
        initView();
        initData();
        initListener();
    }

    private void initView() {
        fragmentOne = new FragmentOne();
        FragmentTransaction transaction;
        transaction = getSupportFragmentManager().beginTransaction();
        transaction.replace(R.id.fl_activity_second, fragmentOne, FragmentOne.TAG);
        transaction.commitAllowingStateLoss();
    }

    private void initData() {

    }

    private void initListener() {
        fragmentOne.setOnFragmentBtnClickListener(this);
    }

    @Override
    public void onFragmentOneBtnClick() {
        fragmentTwo = new FragmentTwo();
        fragmentThree = new FragmentThree();
        fragmentThree.setOnThreeClickListener(this);
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        transaction.add(R.id.fl_activity_second, fragmentTwo, FragmentTwo.TAG);
        transaction.add(R.id.fl_activity_second, fragmentThree, FragmentThree.TAG);
        transaction.addToBackStack(null);
        transaction.commitAllowingStateLoss();
    }

    @Override
    public void onFragmentTwoClick() {

    }

    @Override
    public void onFragmentThreeClick() {
        FragmentManager manager = getSupportFragmentManager();
        FragmentTransaction transaction = manager.beginTransaction();
        transaction.remove(fragmentThree);
        transaction.commitAllowingStateLoss();
    }
}
```
 初始化时并没有将事务压栈，FragmentActivity的返回事件会去检查回退栈，如果栈是null或者size==0直接返回，否则将事务弹出栈：
```java
     /**
     * Take care of popping the fragment back stack or finishing the activity
     * as appropriate.
     */
    @Override
    public void onBackPressed() {
        if (!mFragments.getSupportFragmentManager().popBackStackImmediate()) {
            super.onBackPressed();
        }
    }
```
继续跟源码会到这个判断：  
```java
if (mBackStack == null) {
            return false;
        }
        if (name == null && id < 0 && (flags&POP_BACK_STACK_INCLUSIVE) == 0) {
            int last = mBackStack.size()-1;
            if (last < 0) {
                return false;
            }
```
看到，当回退栈为null或者size==0时，返回false，进而触发super.onBackPressed()操作。
回到例子中，点击了fragmentone中的按钮之后，连续add了两个fragment，但是都是同一个事务，同时将事务压栈，此时回退栈的size为1，此时点击返回键，事务弹出，将回到fragmentone页面，而不是fragmenttwo。如果在fragmentthree页面点击按钮，将会remove掉fragmentthree，这样，fragmenttwo就可见了。所以，remove、add、replace等操作都是针对fragment本身的，回退栈是针对事务的。

 