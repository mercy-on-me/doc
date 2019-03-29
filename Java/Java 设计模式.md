# Java 设计模式
## 前言
### 原则
#### 开闭原则
- 开闭原则 :软件实现对扩展开放,对修改关闭.
- 设计模式可以解耦,提供程序的扩展性

#### 依赖导致原则
- 定义 : 高层模块不要依赖底层模块,二者都应该依赖其抽象
- 抽象不应该依赖细节,细节应该依赖抽象
- 针对接口编程,不要针对实现编程:可以实现接口,继承抽象类
- 优点 : 可以减少类之间耦合性,提供系统稳定性,提供代码可读性和可维护性,降低修改程序所造成的风险


### 简单工厂
#### 概述
- 定义 : 由一个工厂对象决定创建出哪一种产品类的实例
- 类型 : 创建型,但不属于 GOF23种设计模式

#### 适用场景
- 工厂类负责创建的对象比较少
- 客户端(应用层)只知道传入工厂类的参数,对于如何创建对象(或者说创建对象的逻辑)不关心

#### 优缺点
##### 优点
- 只需要传入一个正确的参数,就可以获取所需的对象,而无需知道创建细节

##### 缺点
- 工厂类的职责相对过重,增加新的产品需要修改工厂类的逻辑,违背关闭原则
- 无法形成基于继承的等级结构


## 观察者模式
- 一个观察者订阅被观察者(主题),当被观察者发生变化是,就会通知观察者.
- 定义了多个对象之间的一对多的以依赖关系,当一个对象发生变化时,它的所有依赖者就会收到通知

### 3.1 API
- ==**Observable**== : 主题类,所有的主题都要继承这个类
    
    ```java
    /**
     * 这是个可观察(主题)对象
     * 
     *     一个主题的 Observer set 队列可以有多个观察者. 
     *     观察者是实现 Observer 接口的类. 
     *     当一个主题发生变化后,程序会调用 Observable.notifyObservers() 方法通知所有的观察者,并调用观察者的 update 方法.
     *     notification 和线程机制无关,并且完全独立于 Object 的 wait 和 notify 机制.
     *     当一个主题刚被创建的实时,它的观察者列表是空的.
     *     当且仅当两个 Observer 通过 equals 方法比较结果为 true 的时候,这两个 Observer 才相同
     * The order in which notifications will be delivered is unspecified.
     * The default implementation provided in the Observable class will
     * notify Observers in the order in which they registered interest, but
     * subclasses may change this order, use no guaranteed order, deliver
     * notifications on separate threads, or may guarantee that their
     * subclass follows this order, as they choose.
     */
    public class Observable {
        private boolean changed = false;//标识是否主题改变,用在 setChange,clearChange,
        private Vector<Observer> obs;   //自动增长的数组,用于存放 Observer 对象
    
        /** 构造一个没有观察者的主题. */
        public Observable() {
            obs = new Vector<>();
        }
    
        /**
         * 为主题的 Observer 的 set 队列添加一个 Observer.
         *     如果添加的 Observer 相同,那么就会有两个观察者
         * The order in which notifications will be delivered to multiple
         * observers is not specified. See the class comment.
         * 
         * @param   o  要添加的 Observer
         * @throws 如果 Observer 对象为空,则会抛出NullPointerException
         */
        public synchronized void addObserver(Observer o) {
            if (o == null)
                throw new NullPointerException();
            if (!obs.contains(o)) {
                obs.addElement(o);
            }
        }
    
        /**
         * 从 Observer set队列中删除一个 Observer
         *     如果 o 为 null,不会有任何影响,不会报错,也不会删除 Observer
         *     
         * @param   o   要删除的Observer
         */
        public synchronized void deleteObserver(Observer o) {
            obs.removeElement(o);
        }
    
        /**
         * 当调用 setChange() 将主题状态改为以改变后,可调用此方法通知指定的 Observer,
         * 并会在 notifyObservers(Object arg) 方法内调用 Observer.update方法进行
         *    如果无参或者参数有问题的话,并且在 update 中使用了此参数的话,就会报空指针或者其他问题,具体什么问题需要看 update 内怎么实现的.
         */
        public void notifyObservers() {
            notifyObservers(null);
        }
    
        /**
         * 当调用 setChange() 将主题状态改为以改变后,可调用此方法通知指定的 Observer
         *
         * @param   arg   any object. 
         * @see     java.util.Observable#clearChanged()
         * @see     java.util.Observable#hasChanged()
         * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
         */
        public void notifyObservers(Object arg) {
            /*
             * a temporary array buffer, used as a snapshot of the state of
             * current Observers.
             */
            Object[] arrLocal;  //Observer 数组
    
            synchronized (this) {
                /* We don't want the Observer doing callbacks into
                 * arbitrary code while holding its own Monitor.
                 * The code where we extract each Observable from
                 * the Vector and store the state of the Observer
                 * needs synchronization, but notifying observers
                 * does not (should not).  The worst result of any
                 * potential race-condition here is that:
                 * 1) a newly-added Observer will miss a
                 *   notification in progress
                 * 2) a recently unregistered Observer will be
                 *   wrongly notified when it doesn't care
                 */
                if (!changed)
                    return;
                arrLocal = obs.toArray();   // 获取 Observer 数组
                clearChanged(); //将 改变标识(change)设置为 false.
            }
            //循环 Observer 数组,
            for (int i = arrLocal.length-1; i>=0; i--)
                ((Observer)arrLocal[i]).update(this, arg);
        }
    
        /**
         * 删除主题所有的 Observer.
         */
        public synchronized void deleteObservers() {
            obs.removeAllElements();
        }
    
        /**
         * 将主题设置为 <已改变> ,然后 hasChanged 方法将返回 true
         */
        protected synchronized void setChanged() {
            changed = true;
        }
    
        /**
         * Indicates that this object has no longer changed, or that it has
         * already notified all of its observers of its most recent change,
         * so that the <tt>hasChanged</tt> method will now return <tt>false</tt>.
         * This method is called automatically by the
         * <code>notifyObservers</code> methods.
         *
         * @see     java.util.Observable#notifyObservers()
         * @see     java.util.Observable#notifyObservers(java.lang.Object)
         * 将 change 状态设置为 fasle
         * 指示此对象不再更改，或已将其最近的更改通知其所有观察者，因此 hasChanged 方法现在将返回 false 。此方法由 notifyobserver 方法自动调用。
         */
        protected synchronized void clearChanged() {
            changed = false;
        }
    
        /**
         * Tests if this object has changed.
         *
         * @return  <code>true</code> if and only if the <code>setChanged</code>
         *          method has been called more recently than the
         *          <code>clearChanged</code> method on this object;
         *          <code>false</code> otherwise.
         * @see     java.util.Observable#clearChanged()
         * @see     java.util.Observable#setChanged()
         * 
         * 测试此主题是否发生变化
         */
        public synchronized boolean hasChanged() {
            return changed;
        }
    
        /**
         * 返回主题的 Observer 列表长度
         *
         * @return  the number of observers of this object.
         */
        public synchronized int countObservers() {
            return obs.size();
        }
    }
    ```
    
- ==**Observer**== 观察者接口,提供了 update() 方法,所有的观察者都要实现这个接口,并重写 update() 方法

    ```java
    /**
     * A class can implement the <code>Observer</code> interface when it
     * wants to be informed of changes in observable objects.
     *
     * @author  Chris Warth
     * @see     java.util.Observable
     * @since   JDK1.0
     */
    public interface Observer {
        /**
         * This method is called whenever the observed object is changed. An
         * application calls an <tt>Observable</tt> object's
         * <code>notifyObservers</code> method to have all the object's
         * observers notified of the change.
         *
         * @param   o     the observable object.
         * @param   arg   an argument passed to the <code>notifyObservers</code>
         *                 method.
         */
        void update(Observable o, Object arg);
    }
    ```

### 源码