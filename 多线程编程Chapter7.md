1. 线程组：可以把线程归属到某一个线程组中，线程组中可以有线程对象，也可以有线程组，组中还可以有线程，<font color="red">线程组的作用：可以批量的管理线程或者线程组对象，有效地对线程或线程组对象进行组织。</font>
2. 1级关联：就是父对象中有子对象，但并不创建子孙对象。这样的处理可以对零散的线程对象进行有效的组织和规划。<font color="red">线程必须在运行状态才可以接收组的管理。</font>
3. 多级关联：就是父对象中有子对象，子对象中在创建子对象，即出现子孙对象的效果，但是如果线程树结构设计得非常复杂反而会不利于线程对象的管理。
<pre><code>public class RunTest {
    public static void main(String[] args) {
        //先在main组中添一个线程组A,然后在这个A组中添加线程对象Z
        //方法activeGroupCount()和activeCount()的值不是固定的，是系统中环境的一个快照

        ThreadGroup mainGroup = Thread.currentThread().getThreadGroup();
        ThreadGroup group = new ThreadGroup(mainGroup, "A");
        ThreadGroup group1 = new ThreadGroup(mainGroup, "B");
        //额外新创建一个新线程
        MyThread myThread = new MyThread();


        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("runThread!");
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        };

        //加入main组
        Thread a = new Thread(mainGroup, myThread);
        a.start();
        //runnable加入group组
        Thread newThread = new Thread(group, runnable);
        newThread.setName("ZThread");

        //runnable加入group组
        Thread newThread1 = new Thread(group1, runnable);
        newThread.setName("WThread");

        //线程必须启动然后归到组中
        newThread.start();
        newThread1.start();
        ThreadGroup[] listGroup =
                new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];
        Thread.currentThread().getThreadGroup().enumerate(listGroup);

        System.out.println("main中子线程数：" + listGroup.length);

        //main中活动线程的估计数和活动线程组的估计数
        System.out.println(mainGroup.activeCount() + "----" + mainGroup.activeGroupCount());
        //遍历子线程组 得到名字
        for (ThreadGroup childThread : listGroup){
            System.out.println("子线程组名字：" + childThread.getName());
            System.out.println(childThread.activeCount());
        }

        Thread[] listThread1 = new Thread[listGroup[1].activeCount()];
        //把此线程组及其子组中的所有活动线程复制到指定数组中。
        listGroup[1].enumerate(listThread1);
        System.out.println(listThread1[0].getName());

        Thread[] listThread = new Thread[listGroup[0].activeCount()];
        //把此线程组及其子组中的所有活动线程复制到指定数组中。
        listGroup[0].enumerate(listThread);
        System.out.println(listThread[0].getName());
    }
}</code></pre>
 

----------
1. 线程的自动归属：就是自动归到当前线程组中。在实例化一个ThreadGroup线程组X时，如果不指定所属的线程组，则X线程组自动归到当前线程对象所属的线程组中，也就是隐式的在一个线程组中添加了一个子线程组。
<pre><code>
public class Run {
    public static void main(String[] args) {
        //得到当前线程组的名字以及数量
        System.out.println("A处线程：" + Thread.currentThread().getName() + ", 所属线程：" + Thread.currentThread().getThreadGroup().getName() +
                ", 组中有线程组数量：" + Thread.currentThread().getThreadGroup().activeGroupCount());
        //创建新的线程组，但并为归组
        ThreadGroup group = new ThreadGroup("新的组");

        //再次得到当前线程组的名字以及数量
        System.out.println("B处线程：" + Thread.currentThread().getName() + ", 所属线程：" + Thread.currentThread().getThreadGroup().getName() +
                ", 组中有线程组数量：" + Thread.currentThread().getThreadGroup().activeGroupCount());
        //创建一个线程组数组
        ThreadGroup[] threadGroups = new ThreadGroup[Thread.currentThread().getThreadGroup().activeGroupCount()];

        //把此线程组及其子组中的所有活动线程复制到指定数组(threadGroups)中
        Thread.currentThread().getThreadGroup().enumerate(threadGroups);
        for (int i = 0; i < threadGroups.length; i++)
            System.out.println("第一个线程组名称为：" + threadGroups[i].getName());
    }
}</code></pre>

----------
1. 使线程有序
<pre><code>
public class MyThread extends Thread{
    private Object lock;
    private String showChar;
    private int showNumPosition;
    private int printCount = 0;
    volatile private static int addNumber = 1;

    public MyThread(Object lock, String showChar, int showNumPosition) {
        this.lock = lock;
        this.showChar = showChar;
        this.showNumPosition = showNumPosition;
    }

    @Override
    public void run() {
        super.run();
        synchronized (lock){
            while (true){
                if (addNumber % 3 == showNumPosition){
                    System.out.println("ThreadName=" +
                        Thread.currentThread().getName()
                        + " runCount = " + addNumber + " " + showChar);
                    lock.notifyAll();
                    addNumber++;
                    printCount++;
                    if (printCount == 3){
                        break;
                    }
                }else {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}</code></pre>

----------
1. 