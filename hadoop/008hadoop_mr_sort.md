# Hadoop secondary Sort

## 二次排序基本原理

   二次排序的需求是将全量数据分组，然后在组内进行排序，比如我们要实现全国每个省各个城市的雾霾指数由高到低的排序。这就是一个二次排序的需求，在省内对各城市排序，所以排序划分数据的时候要按照省去划分，同组数据再按照城市的雾霾指数排序，这样就实现了二次排序。

   用mapreduce去实现的话就需要自定义一个排序key有两个属性:省和城市的雾霾指数。正常排序按照省和雾霾指数两个字段排序，但是分发数据的时候定制一个partitioner，按照省字段分发数据，而不是使用默认的划分策略（key的hashcode对reducer个数取余）。
## 代码实现
自定义排序Key类型：SortKeyWritable.java

```
package com.hainiubl.hadoop.demo;

import java.io.DataInput;
import java.io.DataOutput;
import java.io.IOException;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.io.WritableComparable;

public class SortKeyWritable implements WritableComparable<SortKeyWritable> {
    private Text first;  //存放省标识
    private LongWritable sencond; //存放省城市雾霾指数

    public SortKeyWritable(){
        this(new Text(),new LongWritable());
    }
    public SortKeyWritable(Text first, LongWritable sencond) {
        this.first = first;
        this.sencond = sencond;
    }

    @Override
    public void write(DataOutput out) throws IOException {
        first.write(out);
        sencond.write(out);
    }

    @Override
    public void readFields(DataInput in) throws IOException {
        first.readFields(in);
        sencond.readFields(in);     
    }

  //倒序
    @Override
    public int compareTo(SortKeyWritable o) {
        int cmp = this.first.compareTo(o.first);
        if(cmp == 0){
            return -this.sencond.compareTo(o.sencond);
        }
        return -cmp;
    }
    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((first == null) ? 0 : first.hashCode());
        result = prime * result + ((sencond == null) ? 0 : sencond.hashCode());
        return result;
    }
    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }
        if (obj == null) {
            return false;
        }
        if (!(obj instanceof SortKeyWritable)) {
            return false;
        }
        SortKeyWritable other = (SortKeyWritable) obj;
        if (first == null) {
            if (other.first != null) {
                return false;
            }
        } else if (!first.equals(other.first)) {
            return false;
        }
        if (sencond == null) {
            if (other.sencond != null) {
                return false;
            }
        } else if (!sencond.equals(other.sencond)) {
            return false;
        }
        return true;
    }
    public Text getFirst() {
        return first;
    }
    public void setFirst(Text first) {
        this.first = first;
    }
    public LongWritable getSencond() {
        return sencond;
    }
    public void setSencond(LongWritable sencond) {
        this.sencond = sencond;
    }

}

```
二次排序mapreuce job类：

```
package com.hainiubl.hadoop.demo;

import java.io.IOException;

import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.NullWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Partitioner;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.util.StringUtils;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class SecondarySortJob extends Configured implements Tool {
    public static class SortMapper extends Mapper<LongWritable,Text,SortKeyWritable,Text>{

        private Text province = new Text();
        private LongWritable num = new LongWritable();
        private SortKeyWritable sortKey = new SortKeyWritable();

        //数据格式
        // 省\t城市\t雾霾指数
        @Override
        protected void map(LongWritable key, Text value,
                Mapper<LongWritable, Text, SortKeyWritable, Text>.Context context)
                throws IOException, InterruptedException {
            String[] fields = StringUtils.split(value.toString(),'\t');
            if(fields.length < 3){
                return;
            }
            province.set(fields[0]);
            num.set(Long.parseLong(fields[2]));
            sortKey.setFirst(province);
            sortKey.setSencond(num);

            context.write(sortKey, value);
        }

    }

    public static class SortReducer extends Reducer<SortKeyWritable,Text,Text,NullWritable>{

        @Override
        protected void reduce(SortKeyWritable arg0, Iterable<Text> values,
                Reducer<SortKeyWritable, Text, Text, NullWritable>.Context context)
                throws IOException, InterruptedException {
            for(Text v : values){
                context.write(v, NullWritable.get());
            }
        }

    }
    public static class SecondaryPartitioner extends Partitioner<SortKeyWritable, Text>{

        @Override
        public int getPartition(SortKeyWritable key, Text value, int numPartitions) {
            // TODO Auto-generated method stub
            return (key.getFirst().hashCode() & Integer.MAX_VALUE) % numPartitions;
        }

    }

    public int run(String[] args) throws Exception {
        // TODO Auto-generated method stub
        if(args.length < 2){
            System.out.println("<input dir> <output dir>");
            return 2;
        }
        Job job = Job.getInstance(getConf(), "SecondarySortJob");
        job.setJarByClass(SecondarySortJob.class);
        job.setMapperClass(SortMapper.class);
        job.setMapOutputKeyClass(SortKeyWritable.class);
        job.setMapOutputValueClass(Text.class);
        job.setReducerClass(SortReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(NullWritable.class);
        job.setPartitionerClass(SecondaryPartitioner.class);
        job.setNumReduceTasks(2);//两个reducer验证

        Path outPath = new Path(args[1]);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, outPath);

        FileSystem fs = FileSystem.get(getConf());
        if(fs.exists(outPath)){
            fs.delete(outPath, true);
            System.out.println("delete : " + outPath.getName());
        }

        return job.waitForCompletion(true) ? 0 :1 ;
    }

    public static void main(String[] args) throws Exception {
        // TODO Auto-generated method stub
        int res = ToolRunner.run(new SecondarySortJob(), args);
        System.exit(res);
        System.out.println("done");

    }

}
```