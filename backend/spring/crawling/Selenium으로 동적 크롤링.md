``` java
    private final String baseUrl = YOUR_BASE_URL;  
  
    public ArrayList<String> crawling(CrawlingRequest request) {  
        WebDriver webDriver = null;  
        ArrayList<String> data = new ArrayList<>();  
        try {  
            final String url = baseUrl + "?keyword=" + request.keyword() + "&startDate="  
                + request.startDate() + "&endDate=" + request.endDate() + "&sources=blog";  
            System.setProperty("webdriver.chrome.driver", "/usr/local/bin/chromedriver");  
//            System.setProperty("webdriver.chrome.driver", "/Users/gundorit/chromedriver");  
  
            ChromeOptions options = new ChromeOptions();  
            options.addArguments(  
                "user-agent=Mozilla/5.0 (Linux; Android 6.0; Nexus 5 Build/MRA58N) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Mobile Safari/537.36");  
  
            webDriver = new ChromeDriver(options);  
            webDriver.get(url);  
  
            WebDriverWait wait = new WebDriverWait(webDriver, Duration.ofMillis(1000));  
            List<WebElement> gElements = webDriver.findElements(By.cssSelector(  
                "g.association_node"  
            ));  
  
            for (WebElement textElement : gElements) {  
                String text = textElement.getAttribute("textContent");  
                if (text != null) {  
                    data.add(text);  
                }  
            }  
  
            return data;  
        } catch (Exception e) {  
            e.printStackTrace();  
            return null;        } finally {  
            if (webDriver != null) {  
                webDriver.quit();  
            }  
        }  
    }
```