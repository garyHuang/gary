# java 笔记

标签（空格分隔）： java

---
java时间时区转换
```
TimeZone destTimeZone = TimeZone.getTimeZone("GMT+0000");  
TimeZone defaultTimeZone = TimeZone.getDefault() ;
Date date = new Date();
Long targetTime = date.getTime() -
defaultTimeZone.getRawOffset() + destTimeZone.getRawOffset();   
System.out.println( new Date( targetTime ) );
```
系统路径的：LOCALAPPDATA,APPDATA 是很重要的软件使用记录文件，很容易造成账号信息泄漏。
删除电脑上软件使用的痕迹，防止更换电脑时，账号等信息泄漏
```
import java.io.File;
import java.util.Map;
import com.haokuaisheng.utils.Helper;
import com.haokuaisheng.utils.TransformUtils;
public class Test00324 {
	public static void main(String[] args) {
		Map<String, String> getenv = System.getenv() ;
		for(String keyStr:getenv.keySet()){
			String value = TransformUtils.toString(getenv.get(keyStr)) ; 
			
			System.out.println( keyStr + "-->" + value );
		}
		
		File f = new File(System.getenv("LOCALAPPDATA"));
		//deleteFile( f ); 
		System.out.println( f.getAbsolutePath());
		

		File appdata = new File(System.getenv("APPDATA"));
		//deleteFile( appdata ); 
		System.out.println( appdata.getAbsolutePath());
	}
	
	static void deleteFile(File file){
		if(!file.exists()){return;}
		if(file.isFile()){
			System.out.println( file.getAbsolutePath() );
			file.delete();
		}else if(file.isDirectory()){
			File[] listFiles = file.listFiles() ;
			if(Helper.isNull(listFiles)){
				System.out.println( file.getAbsolutePath() );
				file.delete();
				return ;
			}
			for(File f : listFiles){
				deleteFile(f); 
			}
		}
	}
}
```


