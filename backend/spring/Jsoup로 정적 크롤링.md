```java
   public CrwalingResponse crawling(
		   final String baseUrl,
		   final CrawlingRequest request) {  
       List<String> data = new ArrayList<>();  

       try {  
           Connection connection = Jsoup.connect(baseUrl + categoryUrl + thisYearMonth);  
           Document document = connection.userAgent(  
    "user-agent=Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Mobile Safari/537.36")  
			.header("scheme", "https")  
			.header("accept", "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8")  
			.header("accept-encoding", "gzip, deflate, br")  
			.header("accept-language", "ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7,es;q=0.6")  
			.header("cache-control", "no-cache")  
			.header("pragma", "no-cache")  
			.header("upgrade-insecure-requests", "1")  
			.get();  
           
           Elements tableRows = document.select(  
".table.table-bordered.table-striped.bri_rank_table tbody tr");  
  
			return data;
       } catch (IOException e) {  
           e.printStackTrace();  
           return null;       }  
   }
```