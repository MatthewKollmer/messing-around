# About

These contents are the preliminary code and data used to identify racial violence victims in historical newspapers. The code is my own, but full disclaimer: I often collaborate with ChatGPT to make corrections if I get errors and I ask it questions about Python libraries I'm not familiar with.

Also, larger files are not hosted here. They can be found in this Box folder: 

[https://uofi.app.box.com/folder/283228321987?s=hhekbeupm9sgkh8nb3dbo7ial4xpioud](https://uofi.app.box.com/folder/283228321987?s=hhekbeupm9sgkh8nb3dbo7ial4xpioud)

If you don't have access to Box folders and you'd like the larger files, please email me: kollmer2@illinois.edu. I will send them to you.

### Sept 3 Update

Here's a step-by-step breakdown of what I've done so far:

1) Using Seguin & Rigby's lynching dataset ([see here](https://journals.sagepub.com/doi/pdf/10.1177/2378023119841780) and [here](https://archive.ciser.cornell.edu/studies/2833/data-and-documentation)), I extracted names of Black victims between 1883 and 1921, and cities where their murders occurred. If full names or cities were missing, I didn't include them in my subset of the data. This resulted in about 450 instances with victims and places named. See my [preprocessing notebook](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/preprocessing.ipynb) for this code.
2) I put the names, cities, and years of occurrence into a pipeline that scrapes Chronicling America search results. This scraping pipeline identifies any pages from the year of occurrence where the victim name and city appears within 100 words of each other. Then it scrapes the newspaper content of all those pages. The scraped contents were saved to separate csv files (separated by victim names). This resulted in about 350 victim clusters and a total of about 19,000 digitized pages. See my [scraping notebook](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/build_newspaper_dataset.ipynb) for this code.
3) Then I did a little bit of post-processing (more to be done yet). I corrected minor OCR errors (added spaces where they were missing between victim names) and created clippings of the newspapers around instances of victim names (the 100 words before and after victim names). See my [post-processing notebook](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/postprocessing.ipynb) for this code.
4) Look to the lynch_clusters directory in my [Box folder](https://uofi.app.box.com/folder/283228321987?s=hhekbeupm9sgkh8nb3dbo7ial4xpioud) to find the results–about 350 .csv files titled after each victim's name, with pages where they–and the city of their murder–was named.

### Sept 10 Update

Over this week, I enriched the lynch_clusters with geolocation data derived from the Viral Texts' [place metadata](https://github.com/ViralTexts/newspaper-metadata) for newspapers. I also used some Python libraries to build maps of the reprints. The code for these activities can be viewed here: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/geolocate_reprints.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/geolocate_reprints.ipynb).

I also explored the data using these interactive maps. This led to an important discovery: there are a large number of false positives in the data (i.e., instances where the victim's name appears, but not in reference to racial violence). I need to review my scraping code and ChronAm's search. I think I'll need to rebuild the dataset with more refinement. I'm thinking of scraping from a more general search with just the victim's name appearing on the page. Then I'll mine the results for instances where city name and other markers of racial violence occur (words like 'lynch', 'posse', 'mob', 'negro', etc.) within a certain range of the victim names.

Anyway that's not a total loss. In rebuilding the dataset, I'll easily be able to expand by some variables we've discussed in meetings, such as the year following the year of incident and victim aliases. Once I've tried to build the dataset with more refinement, I'll rerun these geolocation and mapping scripts, too, and see how things are looking.

### Sept 18 Update

This week I was admittedly more cluttered in my workflow, but here's what I got done:
- I built a larger dataset based solely on victim names (as opposed to victim names plus city names appearing together). I did this by re-scraping Chronicling America by just victim name in the year of the incident and the year following.
- I debugged some things. I noticed my fix_names() and newspaper_clippings() functions needed to be improved. Basically, I added spaces before and after any fixed_names() then used nltk to tokenize the data to make newspaper_clippings() more accurate in its clippings.
- I added new columns to this broader dataset (victim_name, city, state).
- I added a city_mentioned column with binary logic. If city name is mentioned in the clipping, it is labelled 'yes'.
- I added a signal_word_count column. It contains counts of predetermined signal words (words that probably indicate racial violence) as they appear in the clippings.
- Using these last two columns, I've created a subset of the data where city name appears near victim name and/or racial violence words appear near the victim name. This subset contains just over 10,000 hits. It needs to be reviewed, but I anticipate it will contain fewer false positives of racial violence than the data from my first iteration. I'll also need to test my thresholds to see where I can identify the most instances with fewest false positives. What combination of variables–city of occurrence, what signal words–will most accurately indicate racial violence reports? Not sure how I'll deduce that yet, but I'm thinking on it.

All these steps can be viewed in this messy Jupyter notebook: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/build_refine_dataset.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/build_refine_dataset.ipynb)

### Sept 19 Update

In the last day, I've tried to bring it all together. I've added more columns to the new, refined dataset (columns for 'newspaper' and 'reprint_date'). I also remade the maps with the refined dataset. I also went ahead and created a web page to view the map demo. It is password protected. Contact me if you need the password. But the page can be viewed here: [https://matthewkollmer.com/vrt-say-their-names-demo/](https://matthewkollmer.com/vrt-say-their-names-demo/).

Any changes in my process are reflected in the Jupyter notebooks in this repository now, too. I also uploaded the refined data and the maps to the Box folder here: [https://uofi.app.box.com/folder/283228321987?s=hhekbeupm9sgkh8nb3dbo7ial4xpioud](https://uofi.app.box.com/folder/283228321987?s=hhekbeupm9sgkh8nb3dbo7ial4xpioud)

I think the next thing I need to focus on is testing the accuracy of my racial violence detection. Here's a pseudocode version of how I've narrowed the detection:

Each instance labelled a racial violence report:

- contains the victim's name
- was printed in the year of the occurrence or the year after
- either contains the city of the occurrence within 150 words of the victim's name
  OR
- three or more signal words (lynch, mob, posse, negro, shot, hang) within 150 words of the victim's name

So, how accurate are these results? I'm not quite sure yet. I'm also not sure how to test their accuracy other than combing through them. I'm certain they're more accurate than my first iteration (described in Sept 3 and Sept 10 updates above), but perhaps I can add more signal words or essentialize the city of occurrence to improve accuracy? Again, not sure. I'll continue to think on this over the next few days.

### Sept 27 Update

This week was a week of cogitation. Less coding, more reviewing of the data and pondering what to do with it. At the suggestion of Dr. Ward, I took a look at these resources: [https://journals.sagepub.com/toc/anna/694/1](https://journals.sagepub.com/toc/anna/694/1). I've also been reading a copy of James Loewen's _Sundown Towns_. I've been thinking about how to go further than merely mapping this data. What else can I do with it? I think finding ways to showcase it is valuable, but I also want to interrogate it programmatically and derive information from it. One thing I'm considering is creating embeddings of victim names. How are they treated similarly or differently from one another? Perhaps embeddings would help me answer this question. It may also help me find outliers–victims described in semantically different ways. Perhaps I could create a comparative dataset and embeddings with white victims, too.

I'm also thinking about building datasets of newspapers from regions where lynchings occurred and study what they were publishing ahead of these cases. I think this would be entirely exploratory to start, but the goal would be to find patterns of printed texts–maybe other crimes, other racially charged instances, vigilant committees, etc.–that intimate that the populace was more racially charged and likely to commit acts of racial violence. To be clear, I have no idea how exactly I would mine the data to make these kinds of inferences or predictions, but I could try to figure it out.

Anyway, alongside Avery, I also reviewed the data to see how accurately the clippings are capturing instances of racial violence. I did this in a pseudo-programmatic way. Basically, I built a loop to prompt me to review each row and label the clipping as either totally about racial violence, partially about racial violence, not about racial violence at all, or unknown. This process can be reviewed here: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/case_match_review.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/case_match_review.ipynb). I looked at 171 rows and four csv files. A little over 90% of them were totally or partially about their respective racial violence events. So, that's encouraging–it means that the refined dataset I've put together probably isn't cluttered with false positives. But we'll see how it went for Avery, too. And of course, we should probably review far more of the data. There are something like 10,000 rows. A couple hundred of them is still a small sample size.

Oh, almost forgot–a large part of my week was also preoccupied with reviewing Chronicling America's interface and API changes, which are set to be implemented at the end of this year... Ugh, this is a bit of a nightmare. Once they change the API, most of my code will be broken. I'm going to need to change my scraping approaches drastically. I'm looking more closely at downloading their batches of files rather than scraping by URL. It's going to be a different process altogether, but nothing to be done about it. Must plan accordingly.

### Oct 1 Update

Updates: I decided to forge ahead on the local/regional newspapers idea–that is, the plan to identify towns where lynchings occurred and where nearby local or regional newspapers are 1)digitized and 2) show coverage around the same timeframe. Toward those ends, I put together this notebook: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/identify_build_lynching_town_datasets.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/identify_build_lynching_town_datasets.ipynb). Basically, this notebook contains four general steps:

1) finding localities (city, state) in our data where lynchings occurred
2) Geolocating these lynching towns
3) identify lynching towns with digitized newspapers
4) building lynching town subsets

These steps were successful, but I ran into some surprises along the way. Firstly, after reviewing our Black subset of Seguin & Rigby's data, I discovered it contains very few lynchings from the majority of the Southern states?! This surprised me greatly. There are lots of instances from Texas, but not Louisiana, Mississippi, Georgia, Alabama, or Florida. I need to investigate why. I touch on one hypothesis in the notebook (is it because I removed instances without named victims?), but I haven't investigated this anomaly further.

Secondly, I was surprised by how difficult it was to find towns where lynchings occurred and where there was relevant available newspaper data. Now, this part of the process is ongoing, but I had to review by hand the Chron Am map of newspapers ([see here](https://loc.maps.arcgis.com/apps/instant/media/index.html?appid=3c6a392554d545bdb1c083348ef56458&center=-97.5126;39.6376&level=3)) and my own map of lynchings. This process really revealed the limitations of the Chron Am data. As I say in my notebook, "the limitations of Chron Am's data reveals itself here in ways you don't realize looking at the data as rows or lists. [...] the map makes you see the gaps in spaces where surely there were newspapers, but they are not digitized or available." 

Still, I was able to find a few overlapping spaces and cases. I was also able build subsets of these newspapers by scraping Chron Am. Those subsets can be downloaded here: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/lynching_town_newspapers.zip](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/lynching_town_newspapers.zip). They are:

Newspaper: Peninsula Enterprise, Acconomac, VA 
Page: https://chroniclingamerica.loc.gov/lccn/sn94060041/ 
Incident: Magruder Fletcher, Tasley, VA

Newspaper: Maryland Independent, La Plata, MD 
Page: https://chroniclingamerica.loc.gov/lccn/sn85025407/ 
Incident: Joseph Cocking, Port Tobacco, MD

Newspaper: Lexington Intelligencer, Lexington, MO 
Page: https://chroniclingamerica.loc.gov/lccn/sn86063623/ 
Incident: Harry Gates, Lexington, MO

Now, what to do with these newspaper subsets... I'll begin by reading them. And seeing if they mention anything related to their respective cases. That's what's on the docket for the next few days.

### Oct 15 Update

I'm overdue for an entry here. In the past two weeks, I've been busy on a few fronts with the project. It's growing tendrils, this thing. First, there's the lynching town paper analysis and the pipeline for identifying more lynching town papers. I've written some preliminary, exploratory code for studying lynching town papers: [lynch_town_paper_analysis.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/lynch_town_paper_analysis.ipynb). This notebook contains written considerations and visualizations, but to summarize briefly: I've tried embeddings, word frequency stuff, and topic modeling, but I need to anchor my analyses more thoughtfully. And as for a pipeline for identifying more lynching town papers, I'm on it, but I haven't finished a notebook worth uploading quite yet.

Part of my delay in that regard is the tendril with the largest potential: the additions of the Tolnay-Beck Inventory. You may remember from  [identify_build_lynching_town_datasets.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/identify_build_lynching_town_datasets.ipynb) that I noticed a glaring geospatial gap in our data–there were no recorded lynchings from the Deep South. Well, it turns out that was because Seguin/Rigby's data implicitly tried to expand lynching data to a national scale. They did this because the Tolnay-Beck Inventory–an earlier and widely regarded dataset of lynchings–had recorded only lynchings from the Deep South. In turn, I had to find the Tolnay-Beck Inventory. The PIs for VRT were gracious enough to send me a copy they had from 2016. I also reached out to Dr. Amy Bailey at UIC who maintains the dataset today. She sent me the latest version (last updated 2022).

Anyway, this is a tendril with a lot of potential because the Tolnay-Beck Inventory contains about 3,500 more victims relevant to this project. That's ten times as many as the Seguin/Rigby data... I've done some first steps in preparing this data for the scraping process (see [tolnay_beck_additions.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/tolnay_beck_additions.ipynb)). However, it's gonna take some more planning before I scrape all those results. For reference, scraping Chron Am for the Seguin/Rigby data took about nine hours of runtime. If I have to scrape ten times that amount, it'll take about 90 hours. If possible, I'd like to find a workaround. Otherwise, I'll have to scrape the results in batches over the course of a couple weeks.

We'll see. Tomorrow's meeting will hopefully give me more insight. But for the rest of this week, I'll prepare for a presentation on this project in the Data Cultures Lab (an informal DH lab at UIUC). I'll also see if I can finish the lynching town paper identification pipeline. Lots to do! Tendrils, I say. There's also reviewing the results more thoroughly, improving the signal word lists/lexicon, and other stuff. I think this Friday's presentation will be a good opportunity to get all these things in order.

### Oct 17 Update

Tomorrow I have the presentation for the Data Cultures Lab. As part of my preparation, I put together this flowchart:

![Flowchart for DCL Presentation](https://github.com/MatthewKollmer/messing-around/raw/main/vrt_work/say_their_names/dcl_presentation/vrt_flowchart_dcl_presentation.png)

It attempts to describe the entire process thus far. It only omits things I haven't yet added to this repository. You can also go to [this page](https://www.blocksandarrows.com/editor/lISQXJVO6hjT25OB) if you want to interact with the flowchart.

### Oct 25 Update

I guess I should start by mentioning that the DCL presentation went well! I got some good feedback and suggestions on those dangling research questions in my flowchart. The flowchart also proved to be an effective way to describe what I'm doing. I'll come back to it eventually with updates and so forth. Maybe every couple months, I'll try to translate these journal entries into an updated flowchart.

Anyway, I tried to tackle two things this week:

1) Figuring out how to pull Chron Am data for the Tolnay-Beck Inventory without running my computer for 90+ hours; and,
2) Creating a pipeline for identifying newspapers with digitized content near the locations of lynchings AND over the same timeframe.

On the first task, I've gotten in touch with Dr. Smith at Northeastern. He suggested using the Discovery Cluster/remote server there. It has all the Chron Am data locally. I need to get sponsored access to use this resource at Northeastern, though, so we're filling out the necessary forms. Once approved, I'm hoping he and I can meet to discuss how to use the remote server efficiently. I've done it before, but not with much success... I'm not a command line kinda person. But we'll see, with a little guidance I think I can get it done. Then we'll have the Tolnay-Beck Inventory and any related search results as part of our data.

On the second task, I've had more tangible progress. Here's what I did to identify lynching town newspapers computationally:

I cross-referenced of four datasets:
- Our Seguin & Rigby subset of black victims: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/seguin_rigby_data_black_subset_02.csv](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/seguin_rigby_data_black_subset_02.csv)
- DBpedia's place metadata: [https://github.com/ViralTexts/newspaper-metadata/blob/main/places.csv](https://github.com/ViralTexts/newspaper-metadata/blob/main/places.csv)
- Viral Texts' dbpedia metadata for newspapers: [https://raw.githubusercontent.com/ViralTexts/newspaper-metadata/refs/heads/main/series.csv](https://raw.githubusercontent.com/ViralTexts/newspaper-metadata/refs/heads/main/series.csv)
- Chronicling America's digitized newspaper data: [https://chroniclingamerica.loc.gov/newspapers.txt](https://chroniclingamerica.loc.gov/newspapers.txt)

Using these datasets, I did the following:

1) I cross-referenced VT's dbpedia data with Chron Am's digitized newspaper data. Where there were matches, I added the dbpedia link for location to the digitized newspaper data.
2) I cross-referenced these dbpedia links to the dbpedia places metadata. Where there were matches, I added the dbpedia latitude and longitude data. This gave me the lat/long for each digitized newspaper.
3) After lots of Googling and inquiring with ChatGPT, I focused on what's called the Haversine formula for getting distances between newspaper locations and lynching town locations. This formula calculates the distances between points on a sphere. I'm not an expert on it or anything, but basically this formula calculates the angle between two points and the center of the sphere. Then it multiplies the angle by the size of the sphere to get the distance over the curved surface between the two points. After reading about this method, I adapted code from this Stack Overflow thread: [https://stackoverflow.com/questions/4913349/haversine-formula-in-python-bearing-and-distance-between-two-gps-points](https://stackoverflow.com/questions/4913349/haversine-formula-in-python-bearing-and-distance-between-two-gps-points). This resulted in the function haversine_calculation().
4) Using this function, I iterated over the lats/longs in my newspapers dataframe (the one enriched with location data through the processes in steps 1 and 2) and I iterated over the lats/longs in our Seguin & Rigby Black victim subset. If their locations were within 10 miles of each other, I considered them matches. This 10 mile threshold is just off vibes. It should probably be adjusted based on any information we have about the distance of newspaper circulation and more refined definitions of 'local'. But if there were any matches within 10 miles of each other, I combined the rows in a new dataset called 'nearby_papers_cases'. This resulted in 745 newspapers within 10 miles of where lynching occurred.
5) I still needed to deduce whether these newspapers had digitized pages within the same timeframe as the lynching, though. To do that, I iterated over the First Issue Date and Last Issue Date columns, checking to see if the corresponding lynching date landed within those timeframes. After doing this, I was able to identify about 25 lynching cases that have local papers with digitized coverage.
6) I then mapped the results so it's easier to review them.

There are lots of little processing steps in between these things, but I'm trying to be as clear as possible where it matters. For the full breakdown of these steps in code, you can view this notebook: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/identifying_lynch_town_papers_pipeline.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/identifying_lynch_town_papers_pipeline.ipynb)

To view the mapped results, visit this page: [https://matthewkollmer.com/lynch_town_paper_map.html](https://matthewkollmer.com/lynch_town_paper_map.html)

The red dots are the town locations for lynchings. The black circles are the ten mile radii around local papers. When you hover over either dots or circles, you'll get more information about them, including date, victim, nearest newspaper, etc. 

### Nov 3 Update

I don't have a ton of progress to update this week. Getting remote server access at NEU is a slow process. As it currently stands, I'm waiting for a confirmation email to claim my NEU account. It's been several days of waiting. I sent in a ticket to the NEU IT help desk to try to expedite the process. We'll see. Then it'll be a matter of working with Dr. Smith to know exactly what I'm doing with the remote server... Idk, at this rate, it may have been faster to just segment the Tolnay-Beck data and scrape for a total of 90+ hours runtime. That's a hassle, too, but I'd have most of it done by now. I guess Rome wasn't built in a day, or whatever insufferable aphorism you prefer. I will try to be more patient.

In the interim, I worked on the victim name embedding analysis I had imagined weeks ago. I used BERT to get embeddings from the 'clippings' column and visualized the differences between victim name embeddings with Principal Component Analysis (PCA). These steps can be viewed here: [https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/victim_name_embeddings.ipynb](https://github.com/MatthewKollmer/messing-around/blob/main/vrt_work/say_their_names/victim_name_embeddings.ipynb). There's a visualization in that notebook showing the large cluster of Black victims from our Seguin & Rigby data–no surprise there, that these Black victims are treated, for the most part, semantically the same. But I'm wondering what sort of clusters we might observe by comparing Black and white victims. I could segment the data by timeframe, gender, location, all sorts of things, too, to see how victims compare in terms of their semantic differences in our data.

But first I really need to finish curating the data. I think we've got lots of possibilities here. We have good steps toward analysis. The thing to do now is to finish compiling the data.

### Nov 9 Update

I have no code updates worth pushing but I've been working on my ADHO proposal, NEU remote server access, and other forward-thinking tasks.

I'm hoping to have more on my ADHO proposal by tomorrow or Monday. But on the NEU server access, I've been going back and forth in emails for each step with the IT help desk. I think I'm on the last hurdle for getting access to my account, but we'll see. I'm waiting for them to follow up on reinstating my ability to use their two-factor authentication app which is required to claim the sponsored account.

In the meantime, I've been thinking about how to best present this whole project and make it easily replicable for other scholars. Of course, these entries and all my steps are laid out in the order I completed them, but even I'm beginning to lose track of the strands of work that have gone into this exploration. To make things easier in the future, and to maybe create a more presentable repository, I'm thinking of redrafting these steps in more streamlined order. For example, I might as well combine the Seguin & Rigby & Tolnay-Beck datasets into one file. I've also been cleaning and enriching the data across numerous notebooks–some with subsets, others to enrich with lat/long data, others to add columns and so forth. I think it'd be easier to replicate and demonstrate this work if I essentially revise and consolidate the code. 

If I do this, I'll archive this repository, too. I think there's a lot of intellectual value in the documentation here. It's revealing how this project unfolded in chronological order. But that doesn't necessarily mean it will be the clearest way to showcase the work or allow others to adapt or replicate it. If I do this, I'll also try to rewrite the scraping parts to account for the upcoming changes to the Chronicling America API. That way, I'm also working ahead on future challenges for the project.

### Dec 6 Update

It's been far too long since my last update. Apologies to you, process journal. I didn't mean to neglect these entries. I've been plugging away at tasks but forgetting to record them. It's time for me to catch up.

I think the most important news is that we were able to put together a panel proposal for the ADHO conference. Hooray! Our panel includes VRT folks and a group of PhD students from the iSchool working on a similar NLP project with ephemeral materials. Fingers crossed we're admitted to the conference. I'm not gonna lie, I have high hopes. I won't post all our abstracts here, but quite frankly, I think they're all very good. Here's what I wrote:

“Say Their Names: Text-Mining Digital Newspaper Archives for Lynching Reports and Victim Names”

There is a long history of using data-driven methods to study lynchings in the United States. In the 1880s, the Chicago Daily Tribune published lynching reports from across the nation. Using their data, Ida B. Wells-Barnett conducted some of the first major analyses, revealing how white terrorism increased wherever Black citizens gained political power (Wells-Barnett, 1895). Decades later, the NAACP began tabulating its own data and publishing sweeping reports (NAACP, 1919). So did Monroe Work at the Tuskegee Institute. His reports effectively applied statistical data to dispel white justifications for lynchings (McMurry, 1980).

My paper draws on this history of scholarship and activism. Using victim names recorded in two recent lynching records–the Tolnay-Beck Inventory (1995) and the Seguin & Rigby dataset (2019)–I have text-mined the Chronicling America archive and curated a new dataset of over 10,000 lynching reports from 1865 to 1921. My data includes references to roughly 4,000 lynching victims, providing insight into how they were represented at a national scale via the widespread reports of their murders. This paper aims to provide clear documentation of the production of this dataset and to advocate for its ethical use in studying the painful legacy of lynchings in the United States. To that end, I also demonstrate some effective use cases, including geospatially mapping lynching reports, comparing sentiment toward victims via BERT embeddings, and conducting word frequency analyses to understand how reporting changed over time.

References:

- McMurry, L. O. (1980). A Black Intellectual in the New South: Monroe Nathan Work, 1866-1945. Phylon (1960-), 41(4), 333. https://doi.org/10.2307/274858
- Seguin, C., & Rigby, D. (2019). National Crimes: A New National Data Set of Lynchings in the United States, 1883 to 1941. Socius: Sociological Research for a Dynamic World, 5, 2378023119841780. https://doi.org/10.1177/2378023119841780
- Thirty Years of Lynching in the United States, 1889-1918. (1919). NAACP; Internet Archive. https://archive.org/details/thirtyyearsoflyn00nati
- Tolnay, S. E., & Beck, E. M. (1995). A Festival of Violence: An Analysis of Southern Lynchings, 1882 - 1930. University of Illinois Press.
- Wells-Barnett, I. B. (1895). The Red Record: Tabulated Statistics and Alleged Causes of Lynching in the United States (Northern Illinois University). Digital Library. Retrieved November 18, 2024, from https://digital.lib.niu.edu/islandora/object/niu-gildedage%3A23615

We'll see what the reviewers think. Of course, I think it's a worthwhile topic. I think the essential logic of the computational methods is not particularly groundbreaking. It's a simple keyword-focused information retrieval task. But it works, that's what really matters. The analysis of the subsequent data may yield more innovative methods, though. And most of all, I think there's important work to be done in studying the historical travesties of lynchings with real people named, emphasized. This work is at least providing new ways to do that.

Anyway, that's one major hurdle for the project complete. I've also been working on a polished repository which I mentioned in my previous entries. It can be viewed here [https://github.com/MatthewKollmer/say_their_names](https://github.com/MatthewKollmer/say_their_names). As I stated previously, the goal of this new repository is to support step-by-step replication and presentation of the process(es) of building and analyzing this dataset. In some ways, it feels like I'm creating a rough draft/outline for an article about the dataset. I'm compartmentalizing the steps and labelling them in code notebooks ([01_unify_data_sources](https://github.com/MatthewKollmer/say_their_names/blob/main/01_unify_data_sources.ipynb), [02_pull_search_results](https://github.com/MatthewKollmer/say_their_names/blob/main/02_pull_search_results.ipynb), etc.). When it's fully outlined and presented, I'll basically just need to write an overview of the steps and results for a scholarly audience who may use the factual claims derived from my data analyses, or may use the data itself for their own analyses, to build on my work. That's the goal of this repository. It's been straightforward thus far since it's only representing the initial data curation steps. Once I get to the analysis steps, I may need to pause and consider best orders of presentation. But those challenges are on my radar.

In this new repository, you may notice I've gone ahead and started scraping the results from Chronicling America. Yes, it's true. I'm still working on getting the NEU remote server configured, but in the interim, I've chunked the search results and I'm slowly scraping Chron Am. I'm over halfway done already. This will result in a dataset of about 450,000 search hits. Once complete, I'll take code from the notebooks in this repository to refine the data, essentially repeating my steps with minor alterations to create a dataset of victim references with far fewer false positives. So, that's a solid development.

And that's pretty much everything I've been up to over the past few weeks. There's only one week left in the semester now, but I'll try to post again at least once before we take off.


### Dec 19 Update

Here is a link to the Chron Am raw search results data for victims in both Seguin & Rigby and Tolnay-Beck: [https://uofi.box.com/s/dqf9ac3in29hfz0gbrbleqzvkv13w2hs](https://uofi.box.com/s/dqf9ac3in29hfz0gbrbleqzvkv13w2hs)
