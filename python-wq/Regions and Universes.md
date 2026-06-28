  
Simulate  
Alphas  
Learn  
Data  
Labs  
Genius  
Competitions (3)  
Community  
Simulate  
Alphas  
Learn  
Data  
Labs  
Genius  
Competitions (3)  
Community  
Refer a friend  
Notifications  
User menu

Courses  
Documentation  
Operators  
FAQ  
Events  
Glossary  
Documentation  
Next: Getting Started with USA TOPSP500 universe \[Gold\]  
Global Alphas \[Gold\]  
Global simulations aggregate all the stocks in various regions into a single simulation.

Due to the time differences across the globe, these simulations are currently available in delay-1 mode only.

Alphas

Since ASI is a region, which has multiple countries too, you can try alpha ideas that work well in ASI. In addition to your ideas that work in ASI, you can also try other specific ideas or specific datasets which are only available or have higher coverage in a certain country.  
Rather than thinking about alphas that work in specific countries or exchanges, it might be more advantageous to think of ideas that work generally for equities.  
The advantage of a large universe is that you will have greater confidence in the robustness of the alpha if you can pass the submission criteria, as you can see that historically, your idea worked across the globe in all liquid equities\!  
Global Universe

The Global Region universe is about 9000 stocks in total, spanning several countries.  
Neutralization

Right now, neutralization by market will neutralize all stocks by the global mean values, likewise for sector, subindustry and industry neutralization settings.

It might be more meaningful to instead neutralize by country first before neutralizing by any other grouping (concept of double neutralization will be useful here). This is a new neutralization setting that we have added for Global region. This will neutralize all stocks by their country mean values, hence removing country specific risk from your alpha.  
You can also neutralize your GLB alphas against a group of risk factors. Read this page on risk neutralization for more details. If you select one of SLOW/FAST/SLOW\_AND\_FAST as the neutralization setting, the group neutralization setting defaults to market. Hence, it is a good practice to check your risk-neutralized GLB alpha's performance with the group\_neutralize operator and country grouping  
Sub-Geography Sharpe Cutoff for GLB Alphas

Submission Criteria	Threshold  
AMER Sharpe	≥ 1  
APAC Sharpe	≥ 1  
EMEA Sharpe	≥ 1  
GLB Alphas are subject to additional sub-geography Sharpe cutoff criteria. These criteria ensure that your Alpha isn’t just performing well globally but is also delivering consistent returns across all three geographies. The goal is to avoid over-reliance on a single geography and encourage a more balanced contribution to your overall PnL.

Tips to Improve Sharpe Across Geographies

AMER: Focus on high-liquidity equities and consider incorporating earnings-related signals, as the stocks in these markets are highly responsive to earnings announcements.  
APAC: Pay attention to market microstructure data, as price-volume patterns in APAC markets often differ due to shorter market hours and unique market regulations.  
EMEA: Explore macroeconomic indicators, as EMEA stocks are often influenced by geopolitical events and currency fluctuations.  
GLB Alphas also include a breakdown of global PnL by geography in their PnL chart, making it easy to see how much each geography is contributing. This helps you identify imbalances and determine areas for improvement.  
Picture1.png  
Increasing robustness

Try exploring Derived and Alternate Industry Classification datasets. Using group operators using these grouping fields can help make your alpha signals robust

Next: Getting Started with USA TOPSP500 universe \[Gold\]

Characteristics of TOPSP500 universe  
This USA universe covers around 650 instruments.  
This universe includes stocks in TOP500 universe and some additional stocks from S\&P500 (Standard and Poor's 500). This led to higher industry diversification compared to TOP500. Here's a comparison when simulating Alpha=close for the TOPSP500 & TOP500:  
topsp500\_1.png  
topsp500\_2.png  
Due to its good liquidity and smaller universe size, you can tolerate your Alphas to have higher turnover  
This liquid universe offers you a new option to create liquid Alphas, diversifying your submission is always a good idea\!  
Potential steps to Alpha research in TOPSP5000  
Re-simulate past USA TOP3000 & TOP1000 Alphas with good sub-universe Sharpe on SP500.  
Re-simulate TOP500 Alphas on SP500, the difference in composition may lead to different behavior  
You may observe lower Sharpe and returns compare to bigger universe due to its limited stock numbers  
Option, Analyst Alphas may be a good starting point

New universes are onboarded:

USA TOP2000  
ASI TOP500, ASI MINVOL10M  
GLB MINVOL10M  
Those new universes should enable creation of diversified, low correlated signals.

Comparing to popular MINVOL1M and TOP3000 universes, stocks within those universes are having higher average liquidity, indicating lower trading cost, which should be more suitable to create higher turnover signals.

Introductions  
In this documentation, we will introduce a new Universe for Alpha on USA, EUR, ASI regions called ILLIQUID\_MINVOL1M. It’s a unique universe consisting of illiquid equities, defined by our liquidity metrics: minimum volume of $1M.

The illiquid universe offers potential opportunities to capitalize on short term and long term price discrepancies due to its market inefficiency. However, it also implies higher trading costs, as short selling becomes more difficult and obtaining optimal order pricing is challenging due to slippage. For this reason, a new submission test for Alpha builds has been introduced in the ILLIQUID\_MINVOL1M universe.

getting-started-illiquid-universes.png  
New Submission test  
Most Illiquid instruments after cost Sharpe test measures the proportion of after cost performance in an illiquid universe with reference to the original universe. This test ensures that the most illiquid half of the illiquid universe has a minimum required Sharpe after considering the various costs of trading these instruments

Characteristics of the EUR TOP2500 Universe  
The EUR TOP2500 universe expands from the TOP1200, covering around 2500 instruments. This offers a broader area for analysis.

Tips for Success  
Resimulate your TOP1200 Alphas on the new TOP2500 universe.  
Adapt GLB region Alphas to the EUR TOP2500 universe.  
Start with datasets in price volume, analyst, fundamental, option, and short interest categories.  
In addition to sub-universe test, check Alpha's performance on the TOP800 universe to evaluate performance on liquid instruments.  
Apply Double Neutralization:  
With the expanded EUR TOP2500 universe, more instruments are available within each group. This increased number of instruments per group enhances the reliability and robustness of your Alphas.  
Country Neutralization: Remove country-specific risk by neutralizing by country mean values.  
Sector/Industry Neutralization: Further refine by neutralizing by sector or industry.  
Example: new\_group \= group\_cartesian\_product(country, industry), or new\_group \= densify(country) \+ densify(industry)\*100  
Neutralize EUR Alphas against a group of risk factors using group neutralize operators or use SLOW, FAST, SLOW\_AND\_FAST, CROWDING, STATISTICAL neutralization in settings.  
Take advantage of the expanded EUR TOP2500 universe for more reliable and robust Alphas.

REGION SPECIFICS – Things to consider  
With BRAIN you can create alphas on China’s Stock market \- the second largest in the world by capitalization. There are two primary exchanges:

Shanghai Stock Exchange (SSE): \>2000 listed companies, market cap \>7 trillion USD  
Shenzhen Stock Exchange (SZSE): \>2000 listed companies, market cap \>5 trillion USD  
Different submission criteria  
The China market has a high cost of trading, thus requiring higher returns than other regions.

D1 criteria: Sharpe \>= 2.08; Returns \>= 8%; Fitness \>= 1.0  
D0 criteria: Sharpe \>= 3.5; Returns \>= 12%; Fitness \>= 1.5  
Daily trading limit: price can change in 5-10% range based on stock type. The daily price limit does not apply on the first five trading days after an IPO.\[1\]

Short-selling Restriction: both short selling and margin buying is allowed only for eligible "blue chip" stocks with good earnings performance and is only permitted for locally licensed investors.

This implies that the opposite alphas will not flip the performance. Check alpha example with same simulation setting below:

CHN\_1  
CHN region simulation settings  
China alphas are created and simulated on the “Simulate” page. To run your first simulation on China region:

Click on the gear icon under your simulation tab to open the settings panel.  
Select “CHN” in Region drop down menu and click “Apply”  
chn\_2  
The replicate of China Stock Index 1000 on BRAIN  
The CSI 1000 Index is composed of 1,000 small-scale and well-liquid stocks after excluding the constituent stocks of the CSI 800 Index from all A-shares. It comprehensively reflects the stock price performance of small and medium-sized companies in China's A-share market. You can replicate this index on BRAIN platform:

chn3.png  
Formula  
1  
rank(cap) \< 0.6 ? rank(cap) \> 0.1 ? cap : 0 : 0  
Simulation Settings  
Region	Universe	Language	Decay	Delay	Truncation	Neutralization	Pasteurization	Lookback	Max Trade	Max Position  
CHN	TOP2000	Fast Expression	0	0	0	None	On		Off	Off  
Since CSI1000 takes 1000 stocks based on the market cap ranking from 800 to 1800, so we have this range from 0.1 to 0.6.

Getting started  
Although China’s stock market has several unique characteristics, many common research ideas can be good starting point for your China research before you explore specific ideas:  
Technical indicators (Stochastic Oscillator, Relative Strength Index, MACD etc.)  
Fundamental ratios  
The China market has a high cost of trading, thus requiring higher returns than other regions.  
Apart from usual robustness tests such as sub universes, turnover, fitness and weight, there is an additional test exclusive to the China research region: Robust universe test performance: Alphas are considered good if the robust universe component retains at least 40% of the returns and Sharpe of the submission version.  
Since China market has a price limit system, (i.e., the Shenzhen Stock Exchange and Shanghai Stock Exchange resumed the 10% symmetric price limit system on 26 December 1996 \[1\] ), you will observe the fact that the opposite alphas will not flip the performance (i.e., Sharpe, Returns, Fitness) of alphas.

Please see the following example:

Alpha= ts\_returns(close,5) with Delay=”1”, Neutralization \= “industry”, Decay \= “0”, Truncation \= “0.01”, Universe=”TOP3000” will have the performance as follows:

Sharpe=-6.10, Turnover=63.34%, Returns=-72.73%, Margin=-229.10

However, the oppsite alpha:

Alpha= \-ts\_returns(close,5) with Delay=”1”, Neutralization \= “industry”, Decay \= “0”, Truncation \= “0.01”, Universe=”TOP3000”

will have the performance as follows:

Sharpe=1.86, Turnover=62.69%, Returns=22.99%, Margin=73.30

Characteristics of the GLB TOPDIV3000 Universe  
The GLB TOPDIV3000 universe is designed to enhance Alpha generation with better liquidity, broader coverage (around 2,200 instruments), and balanced global diversity:

USA: Slightly over 50%  
Asia & Europe: Roughly equal at \~25% each  
Additional AMR coverage  
Tips for Success  
Test your previous GLB Alphas on this new universe.  
Improved liquidity and broader coverage may allow Alphas that previously failed Sub-Universe tests to succeed.  
Transfer USA, EUR, and ASI Alphas to the GLB TOPDIV3000 universe for better performance.  
Use price-relative metrics (e.g., ratios) instead of raw prices for consistency across instruments.  
Apply Double Neutralization to improve reliability:  
Country Neutralization: Remove country-specific risks by neutralizing against country mean values.  
Sector/Industry Neutralization: Refine further by neutralizing sector or industry factors.  
Example: new\_group \= group\_cartesian\_product(country, industry)  
Select component Alphas from the TOP3000 or MINVOL1M universes and combine them into SuperAlphas under the GLB TOPDIV3000 universe for enhanced performance.

Characteristics of the MEA Region  
The MEA (Middle East and Africa) region groups selected Middle Eastern and North African markets into one research region. It offers access to a distinct set of country exposures that differ from other regions.

You can verify the included countries by simulating an Alpha with Visualization \= ON and reviewing the country breakdown on BRAIN.

The main MEA equity universe, TOP300, contains approximately 300 stocks, making it a relatively small and capacity-sensitive region.

Tips for Success  
Focus on higher-turnover datasets that update frequently. A turnover of around 20% can be reasonable for this region. Data categories to start:  
Price Volume  
Analyst  
Higher update frequency datasets (check with BRAIN Labs)  
Liquidity and capacity are important in MEA because it is relatively small region. Make sure your Alpha’s margins are high enough to justify the trading costs from higher turnover. Check performance with:  
Max Trade \= ON  
Max Pos \= ON  
Avoid over-concentration in less liquid space, check Average Size By Capitalization by simulating an Alpha with Visualization \= ON  
Use neutralization. A good first step is Market neutralization. You can also apply double neutralization such as sector \+ country.  
Diversity in dataset usage is important in MEA. Do not rely only on price-volume patterns. Where possible, combine them with other signals such as fundamental or analyst data to improve robustness.  
Test robustness across different time periods. In particular, compare behavior before and after 2018 using the test period feature.  
This can help you better understand what is driving your signal and build more robust MEA Alphas.

Introduction  
India universe on BRAIN consists of the top 500 stocks by liquidity.

Normal Alpha tests are applied, including Sharpe, fitness, sub-universe Sharpe, IS ladder etc.

Max Trade is not mandatory in IND, and maximum turnover cap is 40%.

IND need to pass an additional robust universe Sharpe test: IND Alpha should have minimum Sharpe of 1, in liquid stocks selected by BRAIN.

Tips for Success  
IND stocks are not included in ASI universes, also many data appear in both ASI and IND, so we encourage retrying your ASI Alphas in IND. Alphas that do well in both universes, may be strong candidates.

Try diverse data categories, while it can lead to better combined performance.

Though maxtrade is optional, we still encourage keeping the investability constrained PnL to have a reasonable performance.

Though max turnover is 40%, however we still encourage to lower the turnover if possible. Having high margin, with turnover lower than 20% is a good practice.