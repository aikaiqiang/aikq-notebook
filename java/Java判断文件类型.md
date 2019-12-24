## java判断文件类型（根据文件内容前几个字节）

1. MIDI (mid)，文件头：4D546864  
2. JPEG (jpg)，文件头：FFD8FF   
3. PNG (png)，文件头：89504E47   
4. GIF (gif)，文件头：47494638   
5. TIFF (tif)，文件头：49492A00   
6. Windows Bitmap (bmp)，文件头：424D   
7. CAD (dwg)，文件头：41433130   
8. Adobe Photoshop (psd)，文件头：38425053   
9. Rich Text Format (rtf)，文件头：7B5C727466   
10. XML (xml)，文件头：3C3F786D6C   
11. HTML (html)，文件头：68746D6C3E   
12. Email [thorough only] (eml)，文件头：44656C69766572792D646174653A   
13. Outlook Express (dbx)，文件头：CFAD12FEC5FD746F    
14. Outlook (pst)，文件头：2142444E   
15. MS Word/Excel (xls.or.doc)，文件头：D0CF11E0   
16. 2007版Excel，文件头：504B03040A0000000000874EE2400000
17. MS Access (mdb)，文件头：5374616E64617264204A   
18. WordPerfect (wpd)，文件头：FF575043   
19. Postscript (eps.or.ps)，文件头：252150532D41646F6265   
20. Adobe Acrobat (pdf)，文件头：255044462D312E   
21. Quicken (qdf)，文件头：AC9EBD8F   
22. Windows Password (pwl)，文件头：E3828596   
23. ZIP Archive (zip)，文件头：504B0304   
24. RAR Archive (rar)，文件头：52617221   
25. Wave (wav)，文件头：57415645   
26. AVI (avi)，文件头：41564920   
27. Real Audio (ram)，文件头：2E7261FD   
28. Real Media (rm)，文件头：2E524D46   
29. MPEG (mpg)，文件头：000001BA   
30. MPEG (mpg)，文件头：000001B3   
31. Quicktime (mov)，文件头：6D6F6F76   
32. Windows Media (asf)，文件头：3026B2758E66CF11

```java
import java.io.FileInputStream;  
  
public class TestDownload {  
  
    public static String bytesToHexString(byte[] src) {  
        StringBuilder stringBuilder = new StringBuilder();  
        if (src == null || src.length <= 0) {  
            return null;  
        }  
        for (int i = 0; i < src.length; i++) {  
            int v = src[i] & 0xFF;  
            String hv = Integer.toHexString(v);  
            if (hv.length() < 2) {  
                stringBuilder.append(0);  
            }  
            stringBuilder.append(hv);  
        }  
        return stringBuilder.toString();  
    }  
  
    public static void main(String[] args) throws Exception {  
        FileInputStream is = new FileInputStream("abc.jpg");  
        byte[] b = new byte[3];  
        is.read(b, 0, b.length);  
        System.out.println(bytesToHexString(b));  
    }  
}  
```
