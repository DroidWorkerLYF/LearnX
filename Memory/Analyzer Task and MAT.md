# Analyzer Task & MAT
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
![](https://github.com/DroidWorkerLYF/LearnX/blob/master/Memory/memory%20monitor.png?raw=true)

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

### 参考文章
[利用Android Studio、MAT对Android进行内存泄漏检测](https://joyrun.github.io/2016/08/08/AndroidMemoryLeak/)