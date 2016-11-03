# 20151118hadoop分析日志统计IP，UV

标签（空格分隔）： hadoop

---

```

package hadoop.gary.bigdata.mapreduce;

import hadoop.gary.bigdata.enums.VisitMistake;

import java.io.IOException;
import java.util.HashSet;
import java.util.Set;

import org.apache.commons.lang.StringUtils;
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

public class PVMapReduce extends Configured implements Tool{
	
	protected static final String UV="UV";
	protected static final String IP="IP";
	
/*Map过程实现 Mapper 接口，
	 泛型说明：
		第一个：LongWritable 为文件行号，默认这个类型 输入的key
		第二个：Text 为每一行的类型 输入的Value
		第三个：为固定Key
		第四个：为访问的ip或者访问的用户ID
	*/
	public static class PVMapper extends Mapper<LongWritable, Text, Text, Text>{
		
		protected static LongWritable output = new LongWritable(1L);
		
		private Text TEXT_UV=new Text( UV );
		private Text TEXT_IP=new Text( IP );
		
		@Override
		protected void map(LongWritable key, Text value, Context context)
				throws IOException, InterruptedException {
			
			String[] split = value.toString().split("\\s"); 
			
			int length = split.length;
			/*统计数组长度 不够 36的*/
			if(length<36){
				context.getCounter(VisitMistake.LENGTH_LT_36).increment(1);
				return;
			}
			/*统计URL为空的*/
			if(StringUtils.isBlank(split[1])){
				
				context.getCounter(VisitMistake
						.URL_IS_NULL).increment(1);
				return;
			}
			/*获取当前登录的用户Id*/
			String guid = split[5] ; 
			if(StringUtils.isBlank(guid)){
				context.getCounter(VisitMistake
						.GUID_IS_NULL).increment(1);
				return;
			}
			/*将当前用户加入到UV中进行统计*/
			context.write(TEXT_UV , new Text(guid));
			String ip = split[13] ; 
			if(StringUtils.isBlank(guid)){
				context.getCounter(VisitMistake
						.IP_IS_NULL ).increment(1);
				return;
			}
			/*将当前IP加入到IP中统计*/
			context.write(TEXT_IP , new Text(ip) ); 
		}
	}
	
	 
	public static class PVReduce extends Reducer<Text, Text, Text, LongWritable>{
		@Override
		public void reduce(Text key, Iterable<Text> iterable,
				Context context)
				throws IOException, InterruptedException {
			Set<String>sets = new HashSet<String>();
			/*加入Set中去除重复的数据*/
			for(Text writable :iterable){
				sets.add( writable.toString() ); 
			}
			context.write(key , new LongWritable(sets.size()));  
		}
		
	}
	
	
	@Override
	public int run(String[] args) throws Exception {
		
		Job job = Job.getInstance(this.getConf(),//
				this.getClass().getSimpleName()//
			);
		
		job.setJarByClass(this.getClass());
		
		Path inPath = new Path(args[0]);
		FileInputFormat.addInputPath(job, inPath);

		job.setMapperClass(PVMapper.class);
		
		job.setNumReduceTasks(1); 
		job.setPartitionerClass(HashPartitioner.class); 
		
		job.setMapOutputKeyClass( Text.class );
		job.setMapOutputValueClass( Text.class );
		
		
		job.setReducerClass(PVReduce.class);
		job.setOutputKeyClass(Text.class);
		job.setOutputValueClass(LongWritable.class);
		
		FileOutputFormat.setOutputPath(job, new Path(args[1])); 
		
		return job.waitForCompletion(true)?0:1;
	}

	

	public static void main(String[] args) throws Exception {
		Configuration configuration = new Configuration();
				int status  = ToolRunner.run(configuration,
				new PVMapReduce(),//
				new String[]{"/opt/txt/2015082818" , "/opt/out/out21"} //
			) ;
			//exit program
		System.exit(status);
	}
	
	
}
```




