---
layout: single
author_profile: true
title: Data Mining - Earnings Call Transcripts
excerpt: "Retrieve transcript from fool.com. Separate by speaker. NLP"
tags: [Conference Call, NLP, BeautifulSoup, Python]
comments: true
toc: true
toc_sticky: true
header:
  teaser: /assets/images/teaser/code.png
---
# Overview

This project focuses on the data mining and cleaning processes on earnings conference call transcripts, with a little bit of Exploratory data analysis (EDA) at the end.
In this project, we will perform following tasks:
+ Use Beautifulsoup to parse html
+ Split transcripts into several sections: Manager presentation, Q&A
+ Separate text by speaker
+ Calculate word count, sentence count, and sentiment using NLTK
+ Retrieve firm stock price return on the day of the earnings call
+ EDA

This is for educational purposes only, please scrape responsibly.{: .notice--danger}

Install the required libraries


```python
import requests
import urllib.request
from bs4 import BeautifulSoup
import unicodedata
import pandas as pd
import numpy as np
#from scraper_api import ScraperAPIClient
```

# Data Mining

**Set up hyperparameters**


```python
user_agent = 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/35.0.1916.47 Safari/537.36'
headers = {'User-Agent': user_agent}
base = 'https://www.fool.com'
data = pd.DataFrame()
```

**Accessing the HTML content from webpage**

The urls to conference call transcripts are located on https://www.fool.com/earnings-call-transcripts/?page={i} with i is the page number. Each page contains urls to 20 transcripts.

![png](/assets/images/Transcript-Fool/page.png)

If we right click on a tiltl(i.e.) and click Inspect, we see where the link for each transcript are.

```HTML
<a href="/earnings/call-transcripts/2019/11/27/cellcom-israel-ltd-cel-q3-2019-earnings-call-trans.aspx" data-id="article-list-hl">Cellcom Israel Ltd (CEL) Q3 2019 Earnings Call Transcript</a>
```

We loop through pages to get transcripts url. We can specify what page to start  (*start_pg*) and how many pages we want to look at (*num_pg*).

For each page, we use **requests.get()** to request the server the content then use **Beautifulsoup** to parse the HTML content. We then use **find_all()** to find all an anchor tag ('a') that have 'data-id' attribute of 'article-list-hl' which contain the urls we need. Lastly, we add the results from **find_all** into a list (*links*).




```python
links = []

start_pg = 128
num_pg = 2

for i in range(start_pg,start_pg+num_pg):
    url = f"https://www.fool.com/earnings-call-transcripts/?page={i}"
    response = requests.get(url,headers=headers)
    soup = BeautifulSoup(response.text, 'html.parser')
    links += soup.find_all('a', {'data-id' : 'article-list-hl'})
```

Next, we look at each item in the list. Since it's still in html format, we extract *href* and *text* sections of the html and add those to a dataframe.


```python
for l in links:
    data = data.append({'url':base+l['href'],
                    'desc':l.text,'html':"Missing"},
                    ignore_index=True)
```

Our data looks as follows. The *html* column is for the html from transcripts' url.


```python
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>desc</th>
      <th>html</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adecoagro SA (AGRO) Q3 2019 Earnings Call Tran...</td>
      <td>Missing</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cellcom Israel Ltd (CEL) Q3 2019 Earnings Call...</td>
      <td>Missing</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Daktronics Inc (DAKT) Q2 2020 Earnings Call Tr...</td>
      <td>Missing</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Global Cord Blood Corporation (CO) Q2 2020 Ear...</td>
      <td>Missing</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Frontline Ltd (FRO) Q3 2019 Earnings Call Tran...</td>
      <td>Missing</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
  </tbody>
</table>
</div>



Now, we can move on to retreiving html for transcripts. We loop through rows in our dataframe, get link to transcript from url columns, retrieve content in html format, look for *span* tag with *class* atribute of *'article-content'*, and lastly add it to html column.

We can filter out rows that already have the html data and only work on the missing ones.

I add sample(1) to avoid denided by server.


```python
data_missing = data[(data['html']=="Missing")]
for index, row in data_missing.iterrows():       
    url = row['url']
    print(f"Retrieving {url}")
    response = requests.get(url)
    soup = BeautifulSoup(response.content, 'html.parser')
    data.at[index, 'html'] = soup.find('span', {'class' : 'article-content'})
print('Finished downloading transcripts in html format')
```

    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/27/adecoagro-sa-agro-q3-2019-earnings-call-transcript.aspx
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/27/cellcom-israel-ltd-cel-q3-2019-earnings-call-trans.aspx
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/27/daktronics-inc-dakt-q2-2020-earnings-call-transcri.aspx
    ...
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/26/hormel-foods-corp-hrl-q4-2019-earnings-call-transc.aspx
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/26/dycom-industries-inc-dy-q3-2020-earnings-call-tran.aspx
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/26/elbit-systems-ltd-eslt-q3-2019-earnings-call-trans.aspx
    Retrieving https://www.fool.com/earnings/call-transcripts/2019/11/26/bitauto-holdings-limited-bita-q3-2019-earnings-cal.aspx
    Finished downloading transcripts in html format


Let's look at the data with html contents


```python
df = data[(data['html']!="Missing")].copy().reset_index(drop=True)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>desc</th>
      <th>html</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adecoagro SA (AGRO) Q3 2019 Earnings Call Tran...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cellcom Israel Ltd (CEL) Q3 2019 Earnings Call...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Daktronics Inc (DAKT) Q2 2020 Earnings Call Tr...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Global Cord Blood Corporation (CO) Q2 2020 Ear...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Frontline Ltd (FRO) Q3 2019 Earnings Call Tran...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
  </tbody>
</table>
</div>



Here is the content of html column in the first row.


```python
df['html'][0]
```




    <span class="article-content">
    <div class="image imgR"><img alt="Logo of jester cap with thought bubble." src="https://g.foolcdn.com/image/?url=https%3A%2F%2Fg.foolcdn.com%2Feditorial%2Fimages%2F549418%2Ftranscripts-logo.png&amp;w=700&amp;op=resize" srcset="https://g.foolcdn.com/image/?url=https%3A//g.foolcdn.com/editorial/images/549418/transcripts-logo.png&amp;w=300&amp;op=resize 300w, https://g.foolcdn.com/image/?url=https%3A//g.foolcdn.com/editorial/images/549418/transcripts-logo.png&amp;w=1000&amp;op=resize 1000w, https://g.foolcdn.com/image/?url=https%3A//g.foolcdn.com/editorial/images/549418/transcripts-logo.png&amp;w=2000&amp;op=resize 2000w">
    <p class="caption">Image source: The Motley Fool.</p>
    </img></div>
    <p><strong>Adecoagro SA</strong>  <span class="ticker" data-id="224958">(<a href="https://www.fool.com/quote/nyse/adecoagro/agro/">NYSE:AGRO</a>)</span><br>Q3 2019 Earnings Call<br/><span id="date">Nov. 14, 2019</span>, <em id="time">9:00 a.m. ET</em></br></p>
    <h2>Contents:</h2>
    <ul>
    <li>Prepared Remarks</li>
    <li>Questions and Answers</li>
    <li>Call Participants</li>
    </ul>
    <h2>Prepared Remarks:</h2>
    <p><strong>Operator</strong></p>
    <p>Good morning, ladies and gentlemen, and thank you for waiting. At this time, we would like to welcome everyone to Adecoagro's Third Quarter 2019 Results Conference Call. Today with us, we have Mr. Mariano Bosch, CEO; Mr. Charlie Boero Hughes, CFO; and Mr. Juan Ignacio Galleano, Investor Relations-Manager. We would like to inform you that this event is being recorded and all participants will be in a listen-only mode. [Operator Instructions]</p>
    <p>Before proceeding, let me mention the forward-looking statements are based on beliefs and assumptions of Adecoagro's management and on information currently available to the company. They involve risks, uncertainties and assumptions because they relate to future events and therefore depend on circumstances that may or may not occur in the future. Investors should understand that general economic conditions, industry conditions and other operating factors could also affect the results of Adecoagro and could cause results to differ materially from those expressed in such forward-looking statements.</p>
    <p>Now I'll turn the conference over to Mr. Mariano Bosch, CEO. Mr. Bosch, you may begin your conference.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Good morning, and thank you for joining Adecoagro's 2019 third quarter results conference. In our Sugar, Ethanol and Energy business, as already discussed in previous releases, we experienced dry weather in Mato Grosso do Sul with registered rains way below average. Needless to say, this has affected yields and future cane development. The impact though has been mitigated by lowering crushing per hour with the purpose of maximizing ethanol production and allow the cane to continue growth.</p>
    <p>Year-to-date, we were able to divert 83% of our total produced TRS into ethanol. This has allowed us to make a much more efficient use of our sugarcane. Actually, hydrous and anhydrous ethanol traded at 17% and 20% premium to sugar during the year. I would like to stress that the increase in ethanol mix was not only driven by lower crushing per hour. Thanks to our industrial and operational enhancements, at the industry level, we were able to genuinely increase our ethanol daily production by 500 cubic meters. This explains why ethanol accounted for 74% of total sugar, ethanol and energy EBITDA.</p>
    <p>It's important to highlight that we were able to fully profit from our ethanol maximization strategy as we built four additional storage ethanol tanks, adding 80,000 cubic meters of storage capacity. Factoring for the price differential between ethanol in June, July vis-a-vis today's prices, the investments has already been more than paid within a year.</p>
    <p>Regarding our five-year plan, we continue on schedule with the most important part that is a sugarcane planting. Dry weather has certainly delayed our crushing plan for 2019 and the beginning of 2020. We have rains now, and weather appears to be normalizing. This is why we feel confident that as of the second quarter of 2020, we will be able to resume crushing activities according to the original plan.</p>
    <p>Moving to our Farming business, political and economic uncertainties in Argentina posed operational challenges. Specifically, after the primary left [Phonetic] over, market sentiment shifted abruptly and relative prices adjusted accordingly. In this line, our latest farmland independent appraisal reflects a 10% reduction year-over-year for our farmland in Argentina. The value for our Brazilian and US dollars [Phonetic] in farmland, however, remained unchanged. As of today, we are in the middle of the planting season, weather by conditions are optimum. All of our operational teams at the fields are fully focused and aligned. We feel confident to face economic uncertainties ahead. Thanks to the attained efficiencies and to the flexibility we have in all of our businesses to divert sales to the domestic or export market, maximizing margins.</p>
    <p>As a reminder, I would like to highlight that adjusted EBITDA for Argentina represents approximately 30% of total consolidated EBITDA. This explains, among other things, why both Bloomberg and JPMorgan changed our risk perception and reclassified the company as bearing Brazilian risk. We are on the right track to conclude another solid fiscal year, generating good returns. As always, we remain focused in execution to further enhance efficiency. I would like to finish by reiterating my gratitude to all the operational and management teams. After all, it is thanks to their daily efforts and hard work that we have become one of the low-cost producers of food and renewable energy.</p>
    <p>Now I will let Charlie walk you through the numbers of the quarter.</p>
    <p><strong>Carlos A. Boero Hughes</strong> -- <em>Chief Financial Officer</em></p>
    <p>Thank you, Mariano. Good morning, everyone. Let's start on Page four with a brief analysis on the rains in Mato Grosso do Sul. As seen on the chart, weather in our cluster, Mato Grosso do Sul continues to be dry. As a matter of fact, registered rains during the nine-month period of 2019 were 43% below the 10-year average and 38% below same period of last year. Furthermore, during the third quarter of 2019, rains were at 67% lower than the 10-year average, allowing us to accelerate the pace of crushing as it can be seen in the following slides.</p>
    <p>During the third quarter of 2019, a total of 3.7 million tons of sugarcane have been crushed, 11% higher than the third quarter of 2018. This is fully explained by the 25% increase affecting milling days, consequence of dry weather. In the table, it is also possible to see that milling per day went down from 52,000 to 46,000 tons. Indeed, dry weather and the draught has negatively impacted cane development. As a way to secure cane availability for this harvest period, we have decided to strategically reduce milling per hour.</p>
    <p>Please turn now to Page six, where I would like to walk you through our agricultural productivity. Dry weather during the nine-month period of 2019 resulted in a 16% reduction in sugarcane yield. At the same time, TRS during the quarter remain unchanged, totaling 143 kilograms per ton. Dry weather should have resulted in higher TRS content. However, the frost forced us to crush young cane, explaining the TRS level. The combination of these two effects resulted in TRS production per hectare of 9.7 tons, 16.6% lower year-over-year. This, as we shall see translated into higher agricultural costs.</p>
    <p>Let's move ahead to Slide seven, where I would like to discuss our production mix. As you can see on the top left chart, during the third quarter of 2019, anhydrous and hydrous ethanol in Mato Grosso do Sul traded at an average price of $0.153 and $0.143 [Phonetic] per pound sugar equivalent, 31.7% and 23.2% premium to sugar, respectively.</p>
    <p>In this context and leveraging from one of our competitive advantages, most of our TRS production during the first nine months of 2019 was diverted toward ethanol. Indeed, 83% of the extracted sugarcane juice went to ethanol and only 17% for sugar. I would like to insist that this is high degree in flexibility constitutes one of our most important competitive advantages, since it allows us to make a more efficient use of our fixed assets. As a result of this strategy, ethanol accounted for 74.5% of the total EBITDA generation in our Sugar, Ethanol and Energy business during the first nine months of 2019, while sugar accounted for only 12.1%.</p>
    <p>Let's please turn to Slide eight, where I would like to discuss quarterly sales. As you can see on the top left chart, ethanol sales volumes increased by 13.7% compared to the same period of last year. As mentioned, this responds to our strategic decision to maximize ethanol production to profit from higher relative prices. Average selling prices during the quarter increased by 8.2%, reaching $0.145 per pound. All in all, net sales reached $81.5 million, marking at 26.4% increase compared to the third quarter of 2018.</p>
    <p>In the case of energy, selling volumes reached 339,000 megawatt hour, marking a 22.1% increase, explained by the large bagasse availability as a result of higher inventories carried from the fourth quarter of 2018. Our decision of burning wood chips from the beginning of the year, coupled with higher crushing activities, which resulted in higher bagasse availability. Average selling prices measured in US dollars were $54 per megawatt hour, marking a 23.6% decrease compared to the same period of last year. Overall, net sales decreased 6.6% compared to the third quarter of 2018, reaching $18.4 million.</p>
    <p>Sugar sales volumes during the quarter were 104,000 tons, 34.1% lower than the third quarter of 2018. Average net selling prices reached $0.116 per pound, 7.4% higher compared to the third quarter of 2018. As a result, net sales reached $29.9 million, 30.4% lower compared to the third quarter of 2018.</p>
    <p>Next, move to Slide nine, where I would like to explain our production cost. As shown on the bottom graph, total production costs excluding depreciation and the impact of the adoption of IFRS 16, for the nine-month period of 2019 reached $0.059 per pound, 6.6% lower year-over-year. Industrial costs were reduced by 21.4% as a result of higher crushing volumes, enhanced industrial efficiencies and the depreciation of the real. At the same time, these positive effects were partially offset by the 6.2% higher agricultural costs, driven by higher harvested area due to lower yields.</p>
    <p>Finally, to conclude with sugar, ethanol and energy business, please turn to Slide 10, where I would like to discuss financial performance. Adjusted EBITDA for the first nine months of the year totaled $197.9 million, marking a 2.6% increase compared to the first nine months of 2018. The main drivers for the increase were lower production costs, coupled with higher unrealized changes in fair value of the biological assets, partially offset by the mark-to-market effects of our derivative hedge positions.</p>
    <p>To strictly focus on the operational performance of the business, it's more accurate to subtract these non-operating results. Once adjusted, total EBITDA for the first nine months of 2019 reached $181.6 million, 15.3% higher compared to the same period of last year. Higher operational margins were mainly driven by lower production costs and the maximization of ethanol production, as previously explained.</p>
    <p>I would now like to move on to the farming business. Please direct your attention to Slide 12. At the end of the third quarter of 2019, Adecoagro began its planting activities for the 2019-'20 harvest year. We expect to plant 239,000 hectors, 4.1% higher than the previous harvest season. This increase is expected to come primarily from a greater leased area, partially offset by a 2.1% decrease in owned land as a result of the sale of Alto Alegre farm during the first quarter of 2019.</p>
    <p>As of the end of October of 2019, a total of 79,200 hectares or 33.4% of the target area has been seeded. We expect to continue planting rice until mid-November and corn and soybean until early January. The wheat crop has developed as expected, and we are preparing for the start of harvest.</p>
    <p>Let's move to Page 13, where I would like to walk you through the financial performance of our Farming and Land Transformation business. Year-to-date, adjusted EBITDA in the Farming and Land Transformation business reached to $55.8 million, $44.6 million or 44.5% lower year-over-year. Lower financial performance is primarily explained by the $26.9 million lower results generated from farm sales, coupled with lower commodity prices. For the Crops business, we generated an adjusted EBITDA of $17.9 million during the first nine months of 2019, 51.7% or $19.1 million lower compared to the same period of last year. This increase is mainly explained by the combination of lower commodity prices, coupled with lower results from the mark-to-market of our commodity hedge position. These results were partially offset by higher yields and lower production costs measured in US dollars.</p>
    <p>In the case of rice business, adjusted EBITDA reached $17.6 million during the nine-month period, 17.2% lower year-over-year. This was mainly explained by lower rice sales as a result of shipment delay, that was finally registered in October 3rd. Regarding our dairy business, higher production and selling volume, coupled with higher average selling prices, were responsible for the increase in financial performance. At the same time, higher selling volumes were driven by the 19.9% increase in our average cow herds as we continue populating our third free-stall facility.</p>
    <p>Lastly, during the first nine months of 2019, the company completed the sale of Alto Alegre farm resulting in an adjusted EBITDA of $9.4 million compared to the results generated by the sale of Rio de Janeiro and Conquista farms during the first nine months of 2018. It represents a 74.1% decrease. Let's now turn to Page 15, which shows the evolution of Adecoagro's consolidated operational and financial performance.</p>
    <p>As shown on the top left chart, net sales in the first nine months of 2019 reached $596 million, 7% higher year-over-year. This is mainly explained by the combination of higher sales in the crop and dairy businesses as a result of higher selling volumes coupled with higher selling milk prices, partially offset by the combined effect of lower sugar selling volumes, coupled with lower sugar, ethanol and crop prices, measured in dollars. Adjusted EBITDA totaled $239 million during the first nine months of 2019, 14.3% lower compared to the same period of last year. As previously explained, positive results in our Dairy and Sugar, Ethanol and Energy businesses were fully offset by the financial performance of our Rice, Crops and Land Transformation businesses.</p>
    <p>To conclude, please turn to Slide 16 to take a look at our net debt position. As you may see in the left chart, our gross indebtedness as of September 30th, 2019, stands at $899 million, while net debt stands at $753 million, 3% or $22 million lower compared to the previous quarter. This evidence, the beginning of a positive free cash flow cycle as most of the investments related to our five-year growth plan have already been deployed, and we are consolidating and ramping up the operations.</p>
    <p>Net debt ratio reached 2.74 times, 8% compared to the previous quarter. We consider our balance sheet to be in a good position considering not only the adequate debt level but also its long-term tenor. And at the same time, we expect the ratio to decrease as we enter the second semester due to the combined effect of lower working capital requirements and higher EBITDA generation. At the same time, liquidity ratio, which is calculated as cash and equivalents plus marketable inventories divided by short-term debt, reached 1.4 in the third quarter of 2019. Any value above one points the full capacity of the company to reduce short-term debt with cash balance and marketable inventories without raising external capital.</p>
    <p>Thank you very much for your time. We are now open to questions.</p>
    <div class="interad">
    <!-- Create container for ad -->
    <!--Regular Ad --> <!--content1_desk-->
    <div class="dfp-ads ad" id="div-content1_desk-28723">
    <script type="text/javascript">
            googletag.cmd.push(function() {
                googletag.display('div-content1_desk-28723');
            });
        </script>
    </div>
    </div><h2>Questions and Answers:</h2>
    <p><strong>Operator</strong></p>
    <p>Thank you. [Operator Instructions] And our first question will come from Thiago Duarte of BTG. Please go ahead, sir.</p>
    <p><strong>Thiago Duarte</strong> -- <em>BTG Pactual -- Analyst</em></p>
    <p>Thank you so much. Good morning, Mariano. Good morning, Charlie. I have two questions. The first one is on the sugarcane crushing volumes. I appreciate the comments on the drought and the intentions to reduce the crushing activity over the next two quarters and hopefully normalize them by the second quarter of next year. So I was just wondering in terms of total crushing for 2020, how much do you expect the drought as well as the frost, that affected the region in Mato Grosso do Sul to impact your capacity to crush at full speed next year, even considering, of course, that rainfalls have sort of improved recently? And in the end of the day, how much you expect in terms of the net effect for the crushing volumes next year?</p>
    <p>The second question is on the land appraisal that you guys revealed from Cushman and Wakefield, we were a bit surprised about the magnitude of the drop in terms of your land portfolio in Argentina. So I was just wondering, the analogy that we're trying to do here is the last time that Argentina faced more controversial, political and economical environment, including export taxes, retentions and things like that, the land market effectively was very active. Adecoagro itself was capable of monetizing a lot of its land bank at the time. I was wondering whether you guys think that it could actually be the case now instead of lower prices as we saw in the independent appraisal. So just wondering how you see that moving forward. Thank you so much.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Hi, Thiago. Thank you for your question. I'm going to take the second part of your question, and then I'm going to ask Renato to go through the first part of your question, so he can be more precise in the answer of sugarcane. So regarding the land operation, the uncertainties in Argentina are high and also the possibility of retentions and export taxes are clear. So that's why the independent appraisal that we did in September right after the parcel that was probably the worst moment for Argentina, came up with this, say, almost 10% reduction of the price of the Argentine farmland.</p>
    <p>As you clearly said, when there were this type of cost capital controls in Argentina as we have today and we expect for a while to continue to have, the market , the liquidity special increased a lot, and we still think the same that this is probably going to be the same case because we have already more questions and more visits to the farms with people looking to put some money into a dollarized asset. So to finish this part, I would agree with what you were saying that we can expect more activity in our farm sales in the beginning of next year or end of this year or in the medium term.</p>
    <p>And then I'm going to ask Renato to answer the first part of your question regarding our total 2020 crushing possibility.</p>
    <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>
    <p>Hi Thiago, thank you for your question. It was already mentioned by Charlie and Mariano that our yields were impacted by the July frost and the drought that had affected our Mato Grosso do Sul cluster. The frost impact was mostly included in the third quarter, when we harvested and processed the total crop in the area and the drought impact is more distributed through the year. However, since we have already push out of frost affected area, we expect that part of the yields reversal will be offset by future sugarcane higher TRS content.</p>
    <p>And as a result, in '19, we expect to maintain a similar amount of sugarcane that was crushed in 2018, and we shouldn't slow down the crushing pace in the first quarter of 2020. However, since our [Indecipherable] are moving properly, we expect to have enough sugarcane, should be working at full capacity from the second quarter onwards, closing the year with a total production slightly higher than this year.</p>
    <p><strong>Thiago Duarte</strong> -- <em>BTG Pactual -- Analyst</em></p>
    <p>Very good, thank you so much.</p>
    <p><strong>Operator</strong></p>
    <p>Our next question will come from Fernanda Cunha of Citibank. Please go ahead.</p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p>Hi, good morning everyone. Thank you for taking my questions. My first one is in terms of capital allocation EO, could you describe or give us a guideline, what is your intentions for the cash generations in Brazil and in Argentina? If you could separate what are your main initiatives with the positive cash flow. Is dividend or any kind of management liability, pay down some debts in Argentina? If you could detail that for us, that would be great.</p>
    <p>The second one is, in Brazil, there has been a lot of initiatives in terms of projects of ethanol based out of corn and also thermal power generation. I was just wanted to hear your thoughts if you have been looking at these projects, if the returns are compatible with your internal thresholds. And then the last question I have is just a follow-up on Thiago's questions in regards to land sales in Argentina. You mentioned there the market seems quite active now. Can you give us some a few color of what would be a land, where regionally you have been seeing more demand or more interest from potential buyers, please? Thank you.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Hi, Fernanda, thank you for your question. Regarding the first part of the question of the cash generation and what's the -- our capital allocation policies, I would start saying that in our EBITDA generation, Argentina is 30% and 70% is coming from Brazil. So that's really one point that 70% of the cash generation is coming from Brazil. Then as we've been mentioning in the last call, this year 2019, is a year where we end up in a negative free cash flow because of the important capex done according to our five-year plan.</p>
    <p>2020 is the year, where we start being positive free cash flow in a consolidated basis. So this positive free cash flow comes because we are generating more in 2020, I mean the previous year, plus that the capex is reduced a lot, probably by more than half. So we are expecting only $50 million or so capex for 2020 in order to complete our five-year plan, as we've been explaining before.</p>
    <p>So in 2020, it's where we will have a good discussion as we've been explaining, and we will make the decision how we see that we are going to start returning the capital to our shareholders, either dividend or a clear policy on dividend or a buyback or that's part of the discussion that we will have and explain by the end of 2020, because in 2021, the year, where we have the really or the relevant free cash flow positive that is coming from the investments that we've been doing through this five-year plan, as we explained in our Adecoagro already [Phonetic]. So that's basically a summary on our capital allocation and our free cash flow generation.</p>
    <p>Then going to the second question, I'm going to try a quick answer, and then I will ask Renato whether if he got something to add to it that is -- that we've been analyzing many of these corn ethanol-based projects. We always have an approach of being the low-cost producers, and for the regions where we are and with the sustainable developed with a sustainable production model that we have, the most efficient thing that we find in Mato Grosso do Sul is through the sugarcane. So we don't see room in our area with the combination of soil and climate that we have today to be better than with the sugarcane production.</p>
    <p>Remember that we have this continuous harvesting. So this allows us to use in the most efficient way all the assets that we currently have. So that's why I don't see even though we've analyzed many of these projects being more efficient, that's what we are doing today. I don't know, Renato, if you want to add something there.</p>
    <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>
    <p>No, no [Indecipherable]</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Okay. So finally, on the third question regarding land sale, we are as you know, we are always marketing most of our farms, the already transformed farms, and that's part of our current activity. And we see more questions since this capital controls were implemented. In general, in all the farms or in the different regions, we don't have one specific region, where we see more interest. So with all Argentina, we are finding and that's mainly local buyers.</p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p>Okay. Thank you very much. Can I just make a follow-up on the first question? So in terms of the cash generation in Argentina, given the capital controls, would it make sense to maybe pay down the debt in Argentina or accelerate some of the projects there? If you could just be more specific on the cash generated in the country, that would be great.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Yes, of course, we are always using the most efficient way to use this cash flow generation. One of the alternatives say today is through paying debt in Argentina. We already have good relative debt for Argentina. So that's an opportunity to reduce that, and that's a way to generate this cash flow or to use this cash flow generated in Argentina.</p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p>Okay. Thank you.</p>
    <p><strong>Operator</strong></p>
    <p>Our next question will come from Lucas Ferreira of JPMorgan. Please go ahead.</p>
    <p><strong>Lucas Ferreira</strong> -- <em>JPMorgan -- Analyst</em></p>
    <p>Hi, good morning everyone. So my first question is a follow-up on the previous questions on capital allocation. Just to clarify, of course, you consider increasing dividends, maybe buybacks, and in terms of projects and maybe M&amp;A, would you consider at all, just to clarify, investing more in Argentina in the next couple of years? And in Brazil, any other project that you see to improve efficiency or production energy that you think would make sense, so in terms of capex going forward?</p>
    <p>And the second question is regarding your views on the ethanol market for Brazil. If you have any views on the outlook for supply demand inventories now in the intercrop, I see that your stocks are up year-to-date, of course, given the production, but what's your thoughts on the pricing going forward? Do you think that there's still more room to improve pricing would be helpful? Thank you.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Okay. Thank you, Lucas. Regarding your -- the first part of your question regarding the capital allocation and M&amp;A projects and investment-- investments in Argentina. As we've been saying, and I repeat it again, we are already at the end of the cycle of completing our five-year plan. We already did most of the investments, there are very few things, remaining things, that are being done in Argentina. We don't expect to put or to add additional things or improvements or additional investments in Argentina, not even relevant in Brazil. So we are focused on this free cash flow generation. And for us, it is very important to start returning these investments to the shareholders. So we don't see any relevant point here coming in, although we, of course, continue to analyze things going around.</p>
    <p>Finally, on the view on the ethanol, yes, we do have a view for the short and the medium term for the ethanol in Brazil. I would like Renato to go deeper into that question. So Renato, can you go through our view on this of the ethanol?</p>
    <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>
    <p>Hi, Lucas, we have a positive view for ethanol, consider that there is still room for prices to increase even further to get to the 7% parity ratio with gasoline. So the average close to 65%. Despite higher inventories due to the anticipation of the crushing activities in the central south region, group ethanol sales remain above last year, with no signal or stagnation due to high prices. In our view, our current demand will be netted us during the off season to balance the S&amp;D, which should reflect in higher ethanol prices.</p>
    <p>Regarding the mid and long term, we are also confident that ethanol will maintain the upward trend observed in 2019, and it should remain traded with a premium over sugar. The continuous growth in the auto side consumption a positive scenario for oil price that is currently above $60 per barrel and the change in the import quota system should keep prices supported in 2020. In addition, the implementation of an overview will help to increase the profitability for ethanol sales, especially in the medium-term when the [Indecipherable] targets become more aggressive. Regarding another view, all of our three mills have already passed in the certification process. I think in the [Indecipherable] period should start in the next few days due to a high amount of long sugarcane and efficiency of our operations, especially in the use of [Indecipherable] sugarcane, high replacement of fertilizer to concentrated in us. New is hetting impressive scores putting all of our views in the top five in terms of efficiency. Consolidating news already in 20.17 [Phonetic] and allowing us to have one of the best new generations per ton of cane in Brazil.</p>
    <p><strong>Lucas Ferreira</strong> -- <em>JPMorgan -- Analyst</em></p>
    <p>Thank you very much.</p>
    <p><strong>Operator</strong></p>
    <p>Our next question is a follow-up from Fernanda Cunha of Citibank. Please go ahead.</p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p>Hi, thank you for taking my questions. Now, I just wanted if you could give some updates on the RenovaBio program? Also, if you are now that some of your mills, I guess, have been have received the certificate, can we still work with the carbon credit sale cost reduction of around $55 [Phonetic] per cubic meter? Or has that changed?</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Renato?</p>
    <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>
    <p>It's easy to project the amount of CBU that will be selling we're going to be selling approximately 1.5% of CBU per cubic meter. The price of CBU is still unknown. I think the projects that we have been using is around $10 CBU.</p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p>Okay, thank you.</p>
    <p><strong>Operator</strong></p>
    <p>This concludes the question-and-answer section. At this time, I would like to turn the floor back to Mr. Bosch for any closing remarks. Please go ahead, sir.</p>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p>Before closing the last earning call of the earnings call of the year, I would like to thank you all for your support and confidence and let you know that we have renewed our commitment to continue with our obsession to create shareholder value. We have a promising 2020 coming ahead and are ready to accept the challenges. So hope seeing you during our next IR event.</p>
    <p><strong>Operator</strong></p>
    <p>[Operator Closing Remarks]</p>
    <p><strong>Duration: 42 minutes</strong></p>
    <h2>Call participants:</h2>
    <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>
    <p><strong>Carlos A. Boero Hughes</strong> -- <em>Chief Financial Officer</em></p>
    <p><strong>Thiago Duarte</strong> -- <em>BTG Pactual -- Analyst</em></p>
    <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>
    <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>
    <p><strong>Lucas Ferreira</strong> -- <em>JPMorgan -- Analyst</em></p>
    <p><a href="https://www.fool.com/quote/agro">More AGRO analysis</a></p>
    <p>Transcript powered by <a href="https://www.alphastreet.com" target="\_blank">AlphaStreet</a></p>
    <p><em>This article is a transcript of this conference call produced for The Motley Fool. While we strive for our Foolish Best, there may be errors, omissions, or inaccuracies in this transcript. As with all our articles, The Motley Fool does not assume any responsibility for your use of this content, and we strongly encourage you to do your own research, including listening to the call yourself and reading the company's SEC filings. Please see our </em><a href="https://www.fool.com/legal/terms-and-conditions/fool-rules"><em>Terms and Conditions</em></a><em> for additional details, including our Obligatory Capitalized Disclaimers of Liability.</em></p>
    <p></p><div id="pitch"></div>
    </span>



# Data Cleaning

Now we have relevant data in html format. We need to convert them into text.

## Get firm information

First, let's get the company name, stock symbol, and report period for each the transcript. We create a function **get_desc** to look at the text then split it by specific strings.


```python
def get_desc(text):
    """ Extract firm name, ticker and earnings quarter
    from link's description
    """
    firm_name, txt_left = text.split(' (',1)
    ticker, txt_left = txt_left.split(') ',1)
    year_quarter = txt_left.split(' Earnings')[0]
    return firm_name, ticker, year_quarter
```

We use **apply** function in **pandas** to use **get_desc** for each row in a column. We will extract the transcript information from *desc* column.


```python
df['desc'].apply(get_desc)[:10]
```




    0                   (Adecoagro SA, AGRO, Q3 2019)
    1              (Cellcom Israel Ltd, CEL, Q3 2019)
    2                 (Daktronics Inc, DAKT, Q2 2020)
    3    (Global Cord Blood Corporation, CO, Q2 2020)
    4                   (Frontline Ltd, FRO, Q3 2019)
    5                   (Uxin Limited, UXIN, Q3 2019)
    6        (Central Garden & Pet Co, CENT, Q4 2019)
    7          (Arco Platform Limited, ARCE, Q3 2019)
    8       (Dick's Sporting Goods Inc, DKS, Q3 2019)
    9              (Veeva Systems Inc, VEEV, Q3 2020)
    Name: desc, dtype: object



## Split up transcript
Next, we need to separate transcripts into different parts. The conference calls have 2 sections, Management Presentation where managers talk about the firm performance and Question and Answer section where managers answer question from analysts.

Since the sections header are in `<h2>` tags. We can use **find_all** to return a list of all `<p>` and `<h2>` tags and then note the location of `<h2>` tag in the list. Lastly, we split the list using those indexes.


```python
def split_sections(html):
    # Find tags
    if type(html) == str:    
        soup = BeautifulSoup(html, 'html.parser')
        content = soup.find_all(['p','h2'])
    else:
        content = html.find_all(['p','h2'])

    # Add location of above tags to a list
    split_idx = []
    for idx, elem in enumerate(content):
        if elem.name == 'h2':
            split_idx.append(idx)

    # Use split_idx to split the list
    sections = [content[i : j] for i, j in zip([0] + split_idx, split_idx + [None])]
    title = sections[0][1].text
    presn = sections[2]
    qa = sections[3]
    participants = sections[4]

    return title, presn, qa, participants
```

Let's test the function using the first record in our data. Since the function returns a list of 4 values, we will look at those seperately.


```python
transcript_html = df['html'][0]
sections = split_sections(transcript_html)
# Title
sections[0]
```




    'Adecoagro SA\xa0 (NYSE:AGRO)Q3\xa02019 Earnings CallNov. 14, 2019, 9:00 a.m. ET'




```python
# Managers Presenation section
sections[1][:5]
```




    [<h2>Prepared Remarks:</h2>,
     <p><strong>Operator</strong></p>,
     <p>Good morning, ladies and gentlemen, and thank you for waiting. At this time, we would like to welcome everyone to Adecoagro's Third Quarter 2019 Results Conference Call. Today with us, we have Mr. Mariano Bosch, CEO; Mr. Charlie Boero Hughes, CFO; and Mr. Juan Ignacio Galleano, Investor Relations-Manager. We would like to inform you that this event is being recorded and all participants will be in a listen-only mode. [Operator Instructions]</p>,
     <p>Before proceeding, let me mention the forward-looking statements are based on beliefs and assumptions of Adecoagro's management and on information currently available to the company. They involve risks, uncertainties and assumptions because they relate to future events and therefore depend on circumstances that may or may not occur in the future. Investors should understand that general economic conditions, industry conditions and other operating factors could also affect the results of Adecoagro and could cause results to differ materially from those expressed in such forward-looking statements.</p>]




```python
# Q&A section
sections[2][:5]
```




    [<h2>Questions and Answers:</h2>,
     <p><strong>Operator</strong></p>,
     <p>Thank you. [Operator Instructions] And our first question will come from Thiago Duarte of BTG. Please go ahead, sir.</p>,
     <p><strong>Thiago Duarte</strong> -- <em>BTG Pactual -- Analyst</em></p>,
     <p>Thank you so much. Good morning, Mariano. Good morning, Charlie. I have two questions. The first one is on the sugarcane crushing volumes. I appreciate the comments on the drought and the intentions to reduce the crushing activity over the next two quarters and hopefully normalize them by the second quarter of next year. So I was just wondering in terms of total crushing for 2020, how much do you expect the drought as well as the frost, that affected the region in Mato Grosso do Sul to impact your capacity to crush at full speed next year, even considering, of course, that rainfalls have sort of improved recently? And in the end of the day, how much you expect in terms of the net effect for the crushing volumes next year?</p>]




```python
# Participants
sections[3]
```




    [<h2>Call participants:</h2>,
     <p><strong>Mariano Bosch</strong> -- <em>Co-Founder, Chief Executive Officer and Director</em></p>,
     <p><strong>Carlos A. Boero Hughes</strong> -- <em>Chief Financial Officer</em></p>,
     <p><strong>Thiago Duarte</strong> -- <em>BTG Pactual -- Analyst</em></p>,
     <p><strong>Renato Junqueira-Santos Pereira</strong> -- <em>Director of the Sugar, Ethanol and Energy Department.</em></p>,
     <p><strong>Fernanda Cunha</strong> -- <em>Citibank -- Analyst</em></p>,
     <p><strong>Lucas Ferreira</strong> -- <em>JPMorgan -- Analyst</em></p>,
     <p><a href="https://www.fool.com/quote/agro">More AGRO analysis</a></p>,
     <p>Transcript powered by <a href="https://www.alphastreet.com" target="\_blank">AlphaStreet</a></p>,
     <p><em>This article is a transcript of this conference call produced for The Motley Fool. While we strive for our Foolish Best, there may be errors, omissions, or inaccuracies in this transcript. As with all our articles, The Motley Fool does not assume any responsibility for your use of this content, and we strongly encourage you to do your own research, including listening to the call yourself and reading the company's SEC filings. Please see our </em><a href="https://www.fool.com/legal/terms-and-conditions/fool-rules"><em>Terms and Conditions</em></a><em> for additional details, including our Obligatory Capitalized Disclaimers of Liability.</em></p>,
     <p></p>]



## Separate by speaker

To separate a transcript by speaker, we create a function that similar to **split_section**. We look for items with `<strong>` tag and use their location to split.


```python
def seperate_speaker(section):
    split_idx = []
    for idx, elem in enumerate(section):
        if elem.strong:
            #print(elem)
            split_idx.append(idx)
    result = [section[i : j] for i, j in zip([0] + split_idx, split_idx + [None])]
    return result
```

After splitting a transcript by speaker, we create a function **get_speaker_info** to get speakers' information and content of their speech.

Since `<strong>` tag contain speaker's name, job title and firm, we can separate them by spliting using *'--'*. However, if a speaker is from the presenting firm, then the firm name is omitted. Hence, we need to account for that using **if, else** statement.
Lastly, the function returns a dataframe.


```python
def get_speaker_info(section, label):
    df = pd.DataFrame()
    for elem in seperate_speaker(section):
        speaker_name = speaker_firm = speaker_position = ""
        if len(elem) > 1:
            speaker_text = " ".join(map(lambda x: x.text, elem[1:]))
            speaker_info= elem[0].text.split('--')
            if len(speaker_info) == 3:
                speaker_name  = speaker_info[0]
                speaker_firm  = speaker_info[1]
                speaker_position  = speaker_info[2]
            elif len(speaker_info) == 2:
                speaker_name  = speaker_info[0]
                speaker_firm  = "Presenting Firm"
                speaker_position = speaker_info[1]
            else:
                speaker_name = speaker_info[0]

            df = df.append({'Name': speaker_name, 'Position':speaker_position,
                            'Company':speaker_firm, 'Text': speaker_text, 'Section':label}, ignore_index=True)
    df = df[df['Name']!='Operator'].reset_index(drop=True)
    df = df.groupby(['Name','Section'], as_index = False).agg({'Text': ' '.join, 'Position':'first','Company':'first'})
    return df
```

Use **get_speaker_info** function on the presentation and the Q&A sections to further split them up.


```python
presn = sections[1]
qa = sections[2]
qa_by_speaker = get_speaker_info(qa, 'Question and Answer')
presn_by_speaker = get_speaker_info(presn, 'Presentation')

transcript_by_speaker = pd.concat([presn_by_speaker,qa_by_speaker])
transcript_by_speaker
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Section</th>
      <th>Text</th>
      <th>Position</th>
      <th>Company</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Carlos A. Boero Hughes</td>
      <td>Presentation</td>
      <td>Thank you, Mariano. Good morning, everyone. Le...</td>
      <td>Chief Financial Officer</td>
      <td>Presenting Firm</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mariano Bosch</td>
      <td>Presentation</td>
      <td>Good morning, and thank you for joining Adecoa...</td>
      <td>Co-Founder, Chief Executive Officer and Director</td>
      <td>Presenting Firm</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Fernanda Cunha</td>
      <td>Question and Answer</td>
      <td>Hi, good morning everyone. Thank you for takin...</td>
      <td>Analyst</td>
      <td>Citibank</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Lucas Ferreira</td>
      <td>Question and Answer</td>
      <td>Hi, good morning everyone. So my first questio...</td>
      <td>Analyst</td>
      <td>JPMorgan</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mariano Bosch</td>
      <td>Question and Answer</td>
      <td>Hi, Thiago. Thank you for your question. I'm g...</td>
      <td>Co-Founder, Chief Executive Officer and Director</td>
      <td>Presenting Firm</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Renato Junqueira-Santos Pereira</td>
      <td>Question and Answer</td>
      <td>Hi Thiago, thank you for your question. It was...</td>
      <td>Director of the Sugar, Ethanol and Energy Dep...</td>
      <td>Presenting Firm</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Thiago Duarte</td>
      <td>Question and Answer</td>
      <td>Thank you so much. Good morning, Mariano. Good...</td>
      <td>Analyst</td>
      <td>BTG Pactual</td>
    </tr>
  </tbody>
</table>
</div>



## Natural Language Processing

After getting text from all speakers, we can perform some sentiment analysis. Here I used pre-trained Vader analyzer from **NLTK**,


```python
from nltk.tokenize import sent_tokenize, word_tokenize #import sentences and word separator
from nltk.corpus import stopwords # import stop words
from nltk.sentiment.vader import SentimentIntensityAnalyzer
stop_words = set(stopwords.words('english'))
```

The **sent_sentiment** below separate text into sentences, count the number of sentences in the text.  Next, it applies **SentimentIntensityAnalyzer** function from **NLTK** to get the sentiment of a sentence then calculate the average score for all sentences in the text.


```python
def sent_sentiment(text):
    sentences = sent_tokenize(text)
    sent_count = len(sentences)
    scores=[]
    for sentence in sentences:
        ss = SentimentIntensityAnalyzer().polarity_scores(sentence)['compound']
        if ss != 0: scores.append(ss)
    avg_score = np.mean(scores)
    return avg_score, sent_count
```

The **word_count** function count the number of words and stop words in text.


```python
def word_count(text):
    tokens = word_tokenize(text)
    count = len(tokens)
    stop_words_count = len([w for w in tokens if w in stop_words])
    return count, stop_words_count
```

Then we create a function to apply 2 functions above to a dataframe that have a *text* column.


```python
def add_features(df):  
    df['Sentiment'], df['Sent_Count']= zip(*df['Text'].map(sent_sentiment))
    df['Word_Count'], df['Stop_word']= zip(*df['Text'].map(word_count))  
    return df.sort_values("Section")
```


```python
add_features(transcript_by_speaker)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Name</th>
      <th>Section</th>
      <th>Text</th>
      <th>Position</th>
      <th>Company</th>
      <th>Sentiment</th>
      <th>Sent_Count</th>
      <th>Word_Count</th>
      <th>Stop_word</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Carlos A. Boero Hughes</td>
      <td>Presentation</td>
      <td>Thank you, Mariano. Good morning, everyone. Le...</td>
      <td>Chief Financial Officer</td>
      <td>Presenting Firm</td>
      <td>0.120024</td>
      <td>81</td>
      <td>1972</td>
      <td>594</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Mariano Bosch</td>
      <td>Presentation</td>
      <td>Good morning, and thank you for joining Adecoa...</td>
      <td>Co-Founder, Chief Executive Officer and Director</td>
      <td>Presenting Firm</td>
      <td>0.318955</td>
      <td>31</td>
      <td>630</td>
      <td>215</td>
    </tr>
    <tr>
      <th>0</th>
      <td>Fernanda Cunha</td>
      <td>Question and Answer</td>
      <td>Hi, good morning everyone. Thank you for takin...</td>
      <td>Analyst</td>
      <td>Citibank</td>
      <td>0.336576</td>
      <td>24</td>
      <td>374</td>
      <td>153</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Lucas Ferreira</td>
      <td>Question and Answer</td>
      <td>Hi, good morning everyone. So my first questio...</td>
      <td>Analyst</td>
      <td>JPMorgan</td>
      <td>0.428583</td>
      <td>9</td>
      <td>182</td>
      <td>70</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Mariano Bosch</td>
      <td>Question and Answer</td>
      <td>Hi, Thiago. Thank you for your question. I'm g...</td>
      <td>Co-Founder, Chief Executive Officer and Director</td>
      <td>Presenting Firm</td>
      <td>0.384876</td>
      <td>49</td>
      <td>1243</td>
      <td>550</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Renato Junqueira-Santos Pereira</td>
      <td>Question and Answer</td>
      <td>Hi Thiago, thank you for your question. It was...</td>
      <td>Director of the Sugar, Ethanol and Energy Dep...</td>
      <td>Presenting Firm</td>
      <td>0.234414</td>
      <td>20</td>
      <td>562</td>
      <td>213</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Thiago Duarte</td>
      <td>Question and Answer</td>
      <td>Thank you so much. Good morning, Mariano. Good...</td>
      <td>Analyst</td>
      <td>BTG Pactual</td>
      <td>0.220592</td>
      <td>15</td>
      <td>299</td>
      <td>119</td>
    </tr>
  </tbody>
</table>
</div>



## Get Text by section

If we want to compare earnings calls between firms, the dataframe above may not be necessary. Instead, we can just divided transcript's text into 3 parts, management presentation, management Q&A, and analyst Q&A.

The following function uses **get_speaker_info** function above to get text for each section. The argument `analyst=True` is used when we use the function for Q&A section and `False` for Presentation.


```python
def get_text(html, analyst=True):
    if analyst:
        if html == "": return "",""
        df = get_speaker_info(html, label='QA')
        df_a = df[df['Position'].str.contains('Analyst')]
        analyst_text = df_a['Text'].str.cat(sep=' ')
        df_m = df[~df['Position'].str.contains('Analyst')]
        management_text = df_m['Text'].str.cat(sep=' ')
        return analyst_text, management_text
    else:
        if html == "": return ""
        df = get_speaker_info(html, label='Presentation')
        df_m = df[~df['Name'].str.contains('Operator')]
        text = df['Text'].str.cat(sep=' ')
        return text
```

## Retrieve stock data

To get the stock price, we will use **yfinance** package which retrieve prices from Yahoo Finance.


```python
!pip install yfinance --upgrade --no-cache-dir
```

    Requirement already up-to-date: yfinance in /opt/conda/lib/python3.6/site-packages (0.1.54)
    Requirement already satisfied, skipping upgrade: multitasking>=0.0.7 in /opt/conda/lib/python3.6/site-packages (from yfinance) (0.0.9)
    Requirement already satisfied, skipping upgrade: requests>=2.20 in /opt/conda/lib/python3.6/site-packages (from yfinance) (2.22.0)
    Requirement already satisfied, skipping upgrade: pandas>=0.24 in /opt/conda/lib/python3.6/site-packages (from yfinance) (0.25.3)
    Requirement already satisfied, skipping upgrade: numpy>=1.15 in /opt/conda/lib/python3.6/site-packages (from yfinance) (1.18.1)
    Requirement already satisfied, skipping upgrade: idna<2.9,>=2.5 in /opt/conda/lib/python3.6/site-packages (from requests>=2.20->yfinance) (2.8)
    Requirement already satisfied, skipping upgrade: certifi>=2017.4.17 in /opt/conda/lib/python3.6/site-packages (from requests>=2.20->yfinance) (2019.11.28)
    Requirement already satisfied, skipping upgrade: chardet<3.1.0,>=3.0.2 in /opt/conda/lib/python3.6/site-packages (from requests>=2.20->yfinance) (3.0.4)
    Requirement already satisfied, skipping upgrade: urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 in /opt/conda/lib/python3.6/site-packages (from requests>=2.20->yfinance) (1.25.8)
    Requirement already satisfied, skipping upgrade: python-dateutil>=2.6.1 in /opt/conda/lib/python3.6/site-packages (from pandas>=0.24->yfinance) (2.8.1)
    Requirement already satisfied, skipping upgrade: pytz>=2017.2 in /opt/conda/lib/python3.6/site-packages (from pandas>=0.24->yfinance) (2019.3)
    Requirement already satisfied, skipping upgrade: six>=1.5 in /opt/conda/lib/python3.6/site-packages (from python-dateutil>=2.6.1->pandas>=0.24->yfinance) (1.14.0)



```python
import yfinance as yf
import datetime as dt
```

We need 3 arguments to get the stock price: stock ticker, start date, and end date. The `yf.download` will return a dataframe with *open, high, low, close* stock price for a day.

We calculate return as $\frac{Price_{Close} - Price_{Open}}{Price_{Open}}$


```python
yf.download('IBM', start = '2019-12-02', end='2019-12-03')
```

    [*********************100%***********************]  1 of 1 completed





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Open</th>
      <th>High</th>
      <th>Low</th>
      <th>Close</th>
      <th>Adj Close</th>
      <th>Volume</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2019-12-02</th>
      <td>134.449997</td>
      <td>134.5</td>
      <td>132.479996</td>
      <td>132.910004</td>
      <td>131.536484</td>
      <td>3067100</td>
    </tr>
  </tbody>
</table>
</div>



We already have sticker and date for firms in the dataset. However, the date is in `string` not `datetime` object. Therefore, we need to convert string to datetime then convert back to string that follow YYYY-MM-DD format so we can use it in `yf.download`.


```python
def get_stock_price(ticker, date_str):
    # Remove irrelevant string
    date_str = date_str.rsplit(', ', 1)[0]
    date_str = date_str.replace('.','')
    # Convert to datetime
    try:
        date = dt.datetime.strptime(date_str,'%b %d, %Y')
    except:
        date = dt.datetime.strptime(date_str,'%B %d, %Y')
    # Convert back to string with a specific format
    start_date=date.strftime('%Y-%m-%d')  
    end_date = (date +dt.timedelta(days=1)).strftime('%Y-%m-%d')
    # Retrieve stock price
    stock = yf.download(ticker, start=start_date, end=end_date)
    # Calculate return
    price_change = (stock['Close'][0]-stock['Open'][0])/stock['Open'][0]
    return price_change
```

## Preprocessing

Our last function here is to apply all previous functions on the dataset with transcripts in html. We add `try except` in the loop to account for some unexpected issue with the transcript html. If there is error, it will print out the url so we can go and check.


```python
def preprocessing(data):
    df = data.copy()
    df.insert(len(df.columns), 'return', np.nan)
    for idx, row in df.iterrows():
        try:
            # Seperate html into smaller components
            title, presn_html, QA_html, _ = split_sections(row['html'])
            # Add new columns
            df.at[idx,'date'] = title.rsplit('Call', 1)[1]
            df.at[idx,'fname'], df.at[idx,'ticker'], df.at[idx,'yqtr']  = get_desc(row['desc'])

            # Tranform list html to text and combine them
            df.at[idx,'presentation'] = get_text(presn_html, analyst=False)
            df.at[idx,'QA_analyst'], df.at[idx,'QA_mngt'] = get_text(QA_html, analyst=True)
            # Retrieve stock price
            df.at[idx,'return'] = get_stock_price(df.at[idx,'ticker'], df.at[idx,'date'])
        except:
            print('Failed: ', row['url'])
            df.at[idx,'presentation'] = df.at[idx,'QA_analyst'] = df.at[idx,'QA_mngt'] = ''
            continue
    df = df[['fname','ticker','yqtr','date','presentation','QA_analyst','QA_mngt','return']]
    df = df[df['QA_mngt']!='']
    return df.reset_index(drop=True)
```

Here is the input dataset for `preprocessing()`


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>desc</th>
      <th>html</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adecoagro SA (AGRO) Q3 2019 Earnings Call Tran...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cellcom Israel Ltd (CEL) Q3 2019 Earnings Call...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Daktronics Inc (DAKT) Q2 2020 Earnings Call Tr...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Global Cord Blood Corporation (CO) Q2 2020 Ear...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Frontline Ltd (FRO) Q3 2019 Earnings Call Tran...</td>
      <td>[\n, [[\n, &lt;p class="caption"&gt;Image source: Th...</td>
      <td>https://www.fool.com/earnings/call-transcripts...</td>
    </tr>
  </tbody>
</table>
</div>



Here is how our final data looks like.


```python
mydata = preprocessing(df)
mydata.head()
```







<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fname</th>
      <th>ticker</th>
      <th>yqtr</th>
      <th>date</th>
      <th>presentation</th>
      <th>QA_analyst</th>
      <th>QA_mngt</th>
      <th>return</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adecoagro SA</td>
      <td>AGRO</td>
      <td>Q3 2019</td>
      <td>Nov. 14, 2019, 9:00 a.m. ET</td>
      <td>Thank you, Mariano. Good morning, everyone. Le...</td>
      <td>Hi, good morning everyone. Thank you for takin...</td>
      <td>Hi, Thiago. Thank you for your question. I'm g...</td>
      <td>0.034202</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cellcom Israel Ltd</td>
      <td>CEL</td>
      <td>Q3 2019</td>
      <td>Nov 27, 2019, 9:00 a.m. ET</td>
      <td>Yes. Thank you, operator. I would like to welc...</td>
      <td>Hey. Good afternoon. Thanks for taking my ques...</td>
      <td>We're looking very thoroughly to the recommend...</td>
      <td>-0.024735</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Daktronics Inc</td>
      <td>DAKT</td>
      <td>Q2 2020</td>
      <td>Nov 27, 2019, 11:00 a.m. ET</td>
      <td>Thank you, Sheila. Good morning, everyone. As ...</td>
      <td>First one, I just wanted to ask on Live Events...</td>
      <td>Okay. Yes. I think that market will go up and ...</td>
      <td>-0.055639</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Global Cord Blood Corporation</td>
      <td>CO</td>
      <td>Q2 2020</td>
      <td>Nov 27, 2019, 8:00 a.m. ET</td>
      <td>Good morning everyone. In the second quarter, ...</td>
      <td>Yes. Hello, sir, thank you for taking my quest...</td>
      <td>Well, let me first of all apologize, as my est...</td>
      <td>-0.002012</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Frontline Ltd</td>
      <td>FRO</td>
      <td>Q3 2019</td>
      <td>Nov 27, 2019, 9:00 a.m. ET</td>
      <td>Thanks, Robert, and good morning and good afte...</td>
      <td>Sorry, something got there. So, hi, on a dolla...</td>
      <td>I don't -- I think in a way that -- sometimes ...</td>
      <td>-0.017684</td>
    </tr>
  </tbody>
</table>
</div>



# EDA

Let's add 2 new columns for sentiment and sentence count.


```python
mydata['presn_sentiment'], mydata['presn_sent_count'] = zip(*mydata['presentation'].apply(sent_sentiment))
```


```python
import matplotlib.pyplot as plt
```


```python
plt.figure(figsize=(8,10))
plt.subplot(2, 1, 1)
plt.scatter( mydata['return'],mydata['presn_sentiment']);
plt.ylabel('sentiment')
plt.xlabel('return');
plt.subplot(2, 1, 2)
plt.scatter( mydata['return'], mydata['presn_sent_count']);
plt.ylabel('sentence count')
plt.xlabel('return');
```


![png](/assets/images/Transcript-Fool/output_66_0.png)


Based on two plots above, there is no apparent relationship between the change in stock price and sentiment.

For a more accurate test, we can look at correlation table. There is weak correlation between sentiment and return.


```python
mydata[['return','presn_sentiment','presn_sent_count']].corr(method ='pearson')
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>return</th>
      <th>presn_sentiment</th>
      <th>presn_sent_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>return</th>
      <td>1.000000</td>
      <td>0.012025</td>
      <td>-0.095904</td>
    </tr>
    <tr>
      <th>presn_sentiment</th>
      <td>0.012025</td>
      <td>1.000000</td>
      <td>0.136262</td>
    </tr>
    <tr>
      <th>presn_sent_count</th>
      <td>-0.095904</td>
      <td>0.136262</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



## Word cloud

We use the set of stop words from NLTK and add some additional terms into the set.


```python
stop_words = set(stopwords.words('english'))
stop_words.update(['quarter','year','fourth','third','second','first','million','billion'])
```


```python
# Import wordcloud
from wordcloud import WordCloud

# Join text
text = ' '.join(mydata['presentation'].values.tolist())

# Create wordcloud
wordcloud = WordCloud(max_font_size=100, max_words=100,font_step=2,
                      background_color=None, mode='RGBA',colormap ='jet',
                      min_word_length =4, width=1200, height=600, stopwords=stop_words).generate(text)

# Display
plt.figure(figsize=(16,8))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")
plt.title('Wordcloud of Earnings Calls\n', fontdict = {'fontsize' : 30})
plt.show()
```


![png](/assets/images/Transcript-Fool/output_71_0.png)


We can see that *customer, growth, business, market etc.* are the focus of earnings calls.
