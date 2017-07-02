# AKASH-JADHAV-01-07-2017
Implement a Map, which spills on to disk when it exceeds the heap, or a specified limit.  Assumption :  After 4 Entries in map , data is spilled into file




import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;
import java.util.Scanner;
import java.util.Set;

public class Devep {
	
	public static synchronized String get(String key,int lineno) throws IOException{
		File file=new File("map.txt");
		BufferedReader br=new BufferedReader(new FileReader(file));
		String str="";
		if(file.exists() && file.canRead()){
			for(int i=1;i<=lineno;i++){
				str=br.readLine();
			}
			int keyindex=str.indexOf(key);
			int colonindex=str.indexOf(":", keyindex);
			int semicolon=str.indexOf(";",colonindex);
			String value=str.substring(colonindex+1, semicolon);
			System.out.println(value);
			return value;
		}
		return null;
	}
	
	public static void put(String key,String value,int lineno) throws IOException{
		File file=new File("map.txt");
		File file1=new File("map1.txt");
		if(!file.exists()){
			file.createNewFile();
		}
		BufferedReader br=new BufferedReader(new FileReader(file));
		PrintWriter pw=new PrintWriter(new FileWriter(file1));
		if(file.exists() && file.canRead() && file.canWrite()){
			for(int i=1;i<lineno;i++){
				String str=br.readLine();
				if(str == null){
					pw.println();
				}
				else{
					pw.println(str);
				}
			}
			String str=br.readLine();
			if(str == null){
				String entry=((key.concat(":")).concat(value)).concat(";");
				pw.println(entry);
			}
			else if(!str.contains(key)){
				String entry=((key.concat(":")).concat(value)).concat(";");
				str+=entry;
				System.out.println(str);
				pw.println(str);
			}
			else if(str.contains(key)){
				int keyindex=str.indexOf(key);
				int colonindex=str.indexOf(":", keyindex);
				int semicolon=str.indexOf(";",colonindex);
				String old=str.substring(colonindex+1, semicolon);
				str=str.replace(old, value);
				pw.println(str);
			}
			pw.flush();
			pw.close();
			br.close();
			System.out.println(file.delete());
			System.out.println(file1.renameTo(file));
		}
	}

	public static synchronized int lineno(int key){
		return ((key%10)+1);
	}
	
	public static synchronized void overflow(Map<Integer,String> hm) throws IOException{
		Iterator ite=hm.entrySet().iterator();
		while(ite.hasNext()){
			Map.Entry<Integer, String> pair=(Map.Entry<Integer, String>)ite.next();
			put(Integer.toString(pair.getKey()), pair.getValue(),lineno(pair.getKey()));
			ite.remove();                           // removes concurrent modification exception
		}
		Set<Integer> keyset=hm.keySet();
		for(Integer key:keyset){
			hm.remove(key);
		}
	}
	
	public static void main(String[] args) throws IOException {
		Map<Integer,String> hm=new HashMap<>();
		Scanner sc=new Scanner(System.in);
		System.out.println("Enter how many elements wish to insert (Max 4 are allowed in map)");
		int max=sc.nextInt();
		for(int i=0;i<max;i++){
			System.out.println("Enter integer key for "+ (i+1) +" th element");
			int key=sc.nextInt();
			System.out.println("Enter String value for "+ (i+1) +" th element");
			String value=sc.next();
			if(hm.size()==4){
				overflow(hm);
			}
			hm.put(key, value);
		}
		System.out.println("Enter Key to get Value");
		int key=sc.nextInt();
		if(hm.containsKey(key)){
			System.out.println("Value is : " + hm.get(key));
		}
		else{
			String value=get(Integer.toString(key), lineno(key));
			System.out.println("Value is : " + value);
		}
		
	}

}
