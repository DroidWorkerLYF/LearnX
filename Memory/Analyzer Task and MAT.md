# Analyzer Task & MAT
>The Eclipse Memory Analyzer is a fast and feature-rich Java heap analyzer that helps you find memory leaks and reduce memory consumption.

>Use the Memory Analyzer to analyze productive heap dumps with hundreds of millions of objects, quickly calculate the retained sizes of objects, see who is preventing the Garbage Collector from collecting objects, run a report to automatically extract leak suspects.

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
