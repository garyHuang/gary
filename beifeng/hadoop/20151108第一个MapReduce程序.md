# 20151108第一个MapReduce程序

标签（空格分隔）： hadoop

---

第一个MapReduce程序
```java
package hadoop.gary.bigdata.mapreduce;

import java.io.IOException;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.conf.Configured;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.partition.HashPartitioner;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;

public class WordCountMapReduce extends Configured implements Tool{

	/*Map过程实现 Mapper 接口，
	 泛型说明：
	第一个： LongWritable 为文件行号，默认这个类型 输入的key
	第二个： Text 为每一行的类型 输入的Value
	第三个：Text为每个单词的类型 输出的key
	第四个：LongWritable为相同单词的个数 输出的Value
	*/
	public static class WordCountMapper extends Mapper<LongWritable, Text , Text, LongWritable>{

		@Override
		protected void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
				 /*根据 空格 进行拆分单词*/
			String[] split = value.toString().split(" ");
			
			for(String s:split){
				context.write(new Text(s), new LongWritable(1L)); 
			}
		}
		
	}
	
	
/*Reduce过程实现 Reducer 接口，
	 泛型说明：
	第一个： Text 单词，表示Map输入的key
	第二个：LongWritable 单词个数，表示Map输入的值
	第三个：Text 最终返回的key
	第四个：LongWritable最终返回的Value
	*/
	public static class WordCountReduce extends Reducer<Text, LongWritable, Text, LongWritable>{
		@Override
		public void reduce(Text text, Iterable<LongWritable> iterable,
				Context context)
				throws IOException, InterruptedException {
			long sum = 0 ;
			for(LongWritable writable :iterable){
				sum += writable.get();
			}
			context.write(text, new LongWritable(sum));
		}
		
	}
	
	
	/*
	开始执行任务
	*/
	@Override
	public int run(String[] args) throws Exception {
		/*创建一个job任务*/
		Job job = Job.getInstance(this.getConf(),//
				this.getClass().getSimpleName()//
			);
		/*设置当前job运行的class*/
		job.setJarByClass(this.getClass());
		/*获取参数中输入文件路径*/
		Path inPath = new Path(args[0]);
		FileInputFormat.addInputPath(job, inPath);
        
		job.setMapperClass(WordCountMapper.class);
		
		job.setNumReduceTasks(1); 
		job.setPartitionerClass(HashPartitioner.class); 
		
		
		job.setMapOutputKeyClass(Text.class);
		job.setMapOutputValueClass(LongWritable.class);
		
		
		job.setReducerClass(WordCountReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(LongWritable.class);
		
		FileOutputFormat.setOutputPath(job, new Path(args[1])); 
		
		
		return job.waitForCompletion(true)?0:1;
	}



	public static void main(String[] args) throws Exception {
	    /*创建 Configuration 读取配置文件*/
		Configuration configuration = new Configuration();
		/*开始执行任务*/
		int status  = ToolRunner.run(configuration,
				new WordCountMapReduce(),//
				new String[]{"/opt/txt/wc.t" , "/opt/out/out8"} //
			) ;
			/*退出程序*/
			//exit program
			System.exit(status);
	}
	
	
}

```



