# Analyzer Task and MAT
## Analyzer Task
```
public class MainActivity extends AppCompatActivity {
    private Toolbar mToolbar;
    private static Context mLeakActivity;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mToolbar = (Toolbar) findViewById(R.id.tool_bar);
        setSupportActionBar(mToolbar);

        if(mLeakActivity == null){
            mLeakActivity = this;
        }
    }
}
```
这是一段明显会内存泄露的代码。接下来我们通过Analyzer Task分析泄露。
首先触发GC，然后Dump Java Heap。
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Memory_Monitor.png?raw=true)

Android studio会自动打开生成的文件。
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Analyzer_Task_result.png?raw=true)

右侧我们打开Analyzer Task面板，可以看到检测泄露Activity的选项，然后run，就可以得到结果了。选中泄露的Activity可以再Reference Tree中看到更详细的内容。
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Reference_Tree.png?raw=true)

## Eclipse Memory Analyzer(MAT)
>The Eclipse Memory Analyzer is a fast and feature-rich Java heap analyzer that helps you find memory leaks and reduce memory consumption.

>Use the Memory Analyzer to analyze productive heap dumps with hundreds of millions of objects, quickly calculate the retained sizes of objects, see who is preventing the Garbage Collector from collecting objects, run a report to automatically extract leak suspects.

Android studio生成的不是标准hprof文件，我们需要先转换为标准的，然后才能在MAT中打开。
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Export_Standard_Hprof.png?raw=true)
右击要导出的文件，选择Export to standard .hprof，然后指定存储位置即可。

打开刚才生成的文件
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Overview.png?raw=true)

切换到Histogram
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/Histogram.png?raw=true)

可以在第一行中输入要查询的内容，支持正则表达式。上面我们已经知道了泄露了`MainActivity`。接下来借助MAT我们找到引用。输入`MainActivity `，在结果项右击，选择如下的选项。
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/GC_Root.png?raw=true)

结果
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/GC_root_result.png?raw=true)

找到了导致泄露的引用，变量`mLeakActivity `。


### 参考文章
[利用Android Studio、MAT对Android进行内存泄漏检测](https://joyrun.github.io/2016/08/08/AndroidMemoryLeak/)
[Android 性能优化之使用MAT分析内存泄露问题](http://blog.csdn.net/xiaanming/article/details/42396507)