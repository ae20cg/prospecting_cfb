CS 5010: Semester Project Report  
Jesse Katz, Matthew Edwards, Andrej Erkelens  
Group #3  
wgq5tw, me4bb, wsw3fa

## A Data Driven Approach to the NFL Draft 
#### Introduction:
This report examines over 100,000 plays occurring during the 2020 College Football season with the goal to create a process to identify and benchmark strong targets as potential picks in the 2021 and beyond, National Football League (NFL) Drafts. This process is developed through data mining and visualization using multiple different libraries available with Python and utilizes data available through Pro Football Focus. 24 individual players are identified as possible targets based on strong performance on relevant statistics for their position group and these statistics are displayed graphically to assess performance. Data description, experimental design, relevant results, testing, utility of the data and future possibilities are covered.  

#### The Data (and background):
Before diving into the data used to build this draft evaluation process, it is important to understand the connection and significance between college football players and the NFL Draft, the building blocks of a successful NFL roster. In the NFL, each of the 32 teams support a roster with 53 active players. The total salary of all players on the active roster has to fall below the yearly league salary cap, which was set at $198 million in 2020. While the average salary in the NFL is $3.7 million per year, there is a wide distribution of salaries; the highest paid player is the quarterback for the Green Bay Packers: Aaron Rodgers, who is paid $37 million a year. There are only two ways to build a talented roster: the NFL Draft and Free Agency. Free Agency refers to signing a player to the team from another roster when their contracts are up. 

Separately, the NFL Draft refers to an event each year where players eligible for the draft are selected by the NFL teams. Any player coming into the league has to have spent at least 3 years in college (or forfeited their eligibility in some other way). The draft itself consists of seven rounds of selections with each team getting one pick per round. These picks can be traded, and picks are given for compensatory reasons, so the number of picks each team has varies, but is about 7 per team. Once the players are selected, the player signs a contract with the team based on a salary scale. This means that depending on what selection they are, the player is allotted a certain salary by the NFL based on pick tier. These salaries, on average, are much lower than players who are established in the league. This is where the trade offs between two talent recruitment processes fall into play. While signing a player in Free Agency means the team is recruiting a proven commodity in the NFL, they are also typically much more costly. Only signing very strong proven players puts you at risk of going over the salary cap and not being able to field a full team or a team strong at every position. Alternatively, the NFL Draft is a relatively cheap way to build a roster because the rookie salaries are typically much lower than established veteran contracts. What the team gains in salary, they lose in guaranteed and proven performance; many players that are in the draft do not make it more than three years, or do not live up to their potential. Because of this fact, the importance of successfully evaluating and targeting players based on their performance during college is key in building a successful roster. That is what the evaluation process and script described in this report aims to achieve.

The data source utilized in building this process is a database kept by the company Pro Football Focus (PFF) and includes every play that occurred during the 2020 college football season for every game in NCAA Division-I. Some of the information available for each play includes the players involved in the play, play type, play outcome, etc. The data is exclusive to those subscribing (mostly NFL or NCAA teams and staff). This data is kept and published each season in the same format. This allows for easy duplication of any analysis and is discussed later on in this report.   

The PFF dataset, itself, is stored in a comma separated values file with 101,731 rows. Each row represents one play. There are 184 columns in this dataset, each representing specific information or metrics from the play. This ranges from  ball carrier, which indicates which player was running with the football during the play, to the pass result on the play. Each play has its own play ID so it can be identified in the database. A full list of fields can be shown in the attached PDF (PFF Play Feed - Field Definitions 2017) sourced from Pro Football Focuss. A snapshot of the data is shown below:

![Snapshot of Data](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/data_snapshot.PNG?raw=true)

As shown above, each player has a unique identifier. This identifier starts with the abbreviation of the state the player's team is located in. The second two characters are the code for the team the player is on. Then the player's jersey number is shown after a space with two characters. If the player is on defense, the entry has a "D" in front of it. Example is VAUN D06, which represents number 6 on the University of Virginia's defense. Because of the format of the player names, a reference file was created to map in the teams, team conferences, etc. Pro Football Focus stores a PDF of the team codes. The team conferences were taken off of Wikipedia (2020 NCAA Division I FBS football season, 2020) These two supplemental keys were created in csv format for ease of utilization in Python and were named "team_codes.csv" and "team_conferences.csv". Otherwise the dataset itself was relatively clean and not much cleaning was needed. All fields were populated where relevant, and otherwise were coded as "NA". The only other cleaning that was needed was renaming the first field to "plays" because the field had no title but represented a specific play ID, one for each play. Additionally, because each player referenced in the dataset is described by a code, it was known that once the target players for the draft were chosen, they would have to be looked up on the team rosters to add in their names. Lastly, images of team logos were web scraped for visualization purposes from a Github repository CFB (2020). This is discussed in detail

#### Experimental Design
As described above, once the data was retrieved from Pro Football Focus and the supplemental key files were created, the datasets were loaded into a dataframe in Python using the Pandas library (McKinney, W., & others., 2010) and the read_csv function. The minimal data cleaning of adding a column name for the play IDs, the analysis went into a datamining phase where the data was explored for each position group.

The first position group that was explored was defense. It was decided that the key statistics of defensive performance would be the following:  
1. Interceptions
2. Tackles
3. Forced Fumbles  
4. Fumble Recoveries
5. Defensive Touchdowns
6. Sacks
7. Missed Tackles

For each of these key statistics, the dataset was queried, indexed, and sliced to first obtain only plays where the statistic occurred. Then, a smaller dataframe with only the play and the player accumulating that statistic was xreated. One example, is with interceptions where only the play and the player making the interception are included in the dataframe. See the code for this below:

```python
df_interception=df[df['pass_result']=='INTERCEPTION']
df_interception=df_interception[['gsis_play_id','interception']]
```
Once the dataset was reduced, the reduced dataframe was then grouped by the players using groupby and the count() module to obtain the count of the statistic. The player was then set as the index. See an example code below for interceptions:

```python
df_interception_grouped = df_interception.groupby(['interception']).count()
df_interception_grouped=df_interception_grouped.reset_index()
```
This was repeated for each defensive statistic, and then continually merged together on the player names using the merge function in the Pandas library (McKinney, W., & others., 2010). See below for an example:

```python
merged = pd.merge(left=df_tackle_grouped, right=df_interception_grouped, left_on='player', right_on='player', how='left')
```
Once all the defensive statistics were accumulated, the data was further curated and formatted by splitting out the team from player name, using the split method for a string, the apply method and a lambda function. Additional, the true team names and conferences were read into Python and merged into new columns from the supplemental key files. Lastly, because the dataframes were joined on players and some players did not have all the statistics, the null values were replaced using the fillna function with zeroes. 
The last step in the datamining process for the defensive players was to search and narrow down to specific targets. Because different statistics are key indicators of performance for different positions on the defensive side of the football field, different queries were done for different player positions. For secondary players, the data was queried to only find players with at least 2 interceptions, an above average number of tackles, a below average number missed tackles and at least one defensive touchdown. This is shown below:
```python
average_tackles = merged['num_tackles'].mean()
average_missed_tackles = merged['num_missed_tackles'].mean()

defensive_backs = merged[(merged['num_interceptions'] >= 2) & (merged['num_tackles']>= average_tackles)
                         & (merged['num_missed_tackles'] <= average_missed_tackles)
                         & (merged['num_defensive_touchdowns'] >= 1)]
```
This was repeated for pass rushers (if involved in at least fives sacks, less than one missed tackle and at least defensive touchdown or forced fumble) and non-pass rushers (if top five tacklers in a power five conference with less than one missed tackle and at least defensive touchdown or forced fumble). Each position group was assigned a new column specifying their position and each of the total 14 defensive targets were looked up in their team roster to find their full names. These names were also added as a new column.

The next step for the defensive players were to visualize the target performance to see how they ranked against each other. All defensive statistics for each player were plotted on a stacked bar chart using the matplotlib library (2007) shown in the "Results" section and discussed further.

Additionally, two key statistics, sacks and tackles, for pass rushers and tacklers were plotted against each other with the player's team logo as the marker. This was done by web scraping content from the CFB (2020) github repository and saving each logo with the team name as a PNG in a folder in the Python project folder. The webscraping process is discussed in more detail in the "Beyond the original specifications" section. See below for the code that utilizes the OS library (python standard library) to get the working directory and create the folder to store the images:
```python
path = os.getcwd() + '\\Logos\\'
try:
    os.mkdir(path)
except OSError:
    print ("Creation of the directory %s failed" % path)
else:
    print ("Successfully created the directory %s " % path)
#read logos
urls = pd.read_csv('https://raw.githubusercontent.com/spfleming/CFB/master/logos.csv')
for i in range(0,len(urls)):
    try:
        urllib.request.urlretrieve(urls['logo'].iloc[i], os.getcwd() + '\\Logos\\' + urls['school'].iloc[i] + '.png')
    except:
        pass # doing nothing on exception to skip where HTML not found
```
The logos were then pulled and matched up to the target players and plotted using Matplotlib (2007) and the AnnotationBbox function. This plot is shown and discussed further in the "Results" section. 

The next position group that was analyzed was the kicker position. A similar process was followed to reduce the main dataset into a subset with only plays that involved field goals. The field goals made, field goals missed, and their respective yardage for each attempt were retrieved. Then, using the crosstab function in the Pandas (2010) library, the percentage made and total attempts were stored together in a single dataframe for all kickers. The field goal attempt yardage was split into the following ranges (0.0, 20.0], (20.0, 30.0], (30.0, 40.0], (40.0, 50.0] and then greater than 50 years in order to see how the kickers performed on different types of kicks. Only kickers that attempted all kick distances were investigated and the rest were dropped via dropna(). The code to achieve this is shown below where the final dataframe is a multindex object, t3:

```python
t1 = pd.crosstab([df_fg.kicker,pd.cut(df_fg.kick_yards,[0.001,20,30,40,50,np.Inf],include_lowest=True)]
                   ,df_fg.kick_result,normalize='index').reset_index()
t2 = pd.crosstab(index=[df_fg['kicker'], pd.cut(df_fg['kick_yards'],[0, 20, 30, 40, 50, np.inf])], 
            columns=df_fg['kick_result'], 
            margins=True, margins_name='Total_Attempts').reset_index()
t1.set_index('kicker')
t2.set_index('kicker')
t2.drop(540)
del t2['kicker']
del t2['kick_yards']
del t2['MADE']
del t2['MISS']
t3 = pd.concat([t1, t2], axis=1)
t3.head()
t3.set_index('kicker')

#Convert the long data to wide data so that I can access all data based on player
t3 = t3.pivot(index='kicker', columns='kick_yards', values=['MADE','Total_Attempts']) # Convert the long data to wide
t3.head()
t3.columns ## View the columns
t3 = t3.drop('nan', axis=1, level=1) ## Drop these 2 columns in our multi index dataframe
t3 = t3.dropna() ## Attempted every range
```
Similar to defensive players, the dataset was further refined by querying to find targets meeting the following specifications: attempted a FG from every range, 100% accuracy from 0-20 yards, greater than 90% from 20-30 yards, greater than 80% from 30-40 yards, greater than 70% from 40-50 yards, and left up to the team for the over 50 yard attempts. This left 10 kickers that were added together with the defensive targets into a final targets dataframe. The kicker team names, position types, etc. were also included as done for the defensive players.
  
Focusing on the top three kicker prospects, it proved useful to visualize performance and accuracy on a field like plot. A graphic of a football field was created for each of the three prospects and  both the field goal misses and makes were plotted by their position on the field. A function was created that allowed input of the kicker name and then the plot is produced. The plots are shown below in the "Results" section and discussed further for the top three prospects. 

The last position group analyzed was the quarterback group. Similar to what was done for the defensive players and kickers, the dataset was sliced and reduced to only include pass plays and the quaterback involved, as well as various statistics around the play like pass result, pass width, and pass depth. The reduced dataframe was then crosstabulated into a multindex dataframe based on the quarterback and the pass result for that play. Columns were added to provide additional relevant statistics like total passes attempted, completion percentage, and touchdowns using various groupby and other Pandas (2010) methods. See below for this code:
```python
df_passes=df[df['run_pass'] == 'P']
df_passes=df_passes[['play_id','quarterback','pass_result','pass_depth','pass_width', 'touchdown', 'pass_receiver_target', 'incompletion_type']]

df_passes=df_passes[df_passes['pass_result'] != 'RUN']
df_passes=df_passes[df_passes['pass_result'] != 'SACK']

QBDFPasses = pd.crosstab(df_passes['quarterback'], df_passes['pass_result'])

del QBDFPasses['LATERAL']
del QBDFPasses['HIT AS THREW']
QBDFPasses['TotalPasses']= QBDFPasses.sum(axis=1)
QBDFPasses['CompPerc'] = (QBDFPasses['COMPLETE'] / QBDFPasses['TotalPasses']) * 100
QBDFPasses['Touchdowns'] = df_passes.groupby('quarterback').touchdown.count()

QBDFPasses = QBDFPasses[QBDFPasses.TotalPasses >= 200]
QBDFPasses['quarteraback'] = QBDFPasses.index
```
In the 2021 NFL Draft, there were five top quarterback prospects: Trevor Lawrence, Justin Fields, Mac Jones, Zach Wilson and Trey Lance. We analyzed these top prospects, excluding Lance who did not play during the 2020 season due to COVID. One way to visualize quarterback performance is via a scatterplot of results of all their throws based on targeted width and depth. These plots were created for the four prospects as shown below in the "Results" section and discussed further. 
The four quarterback targets were added to the final targets dataframe with the kickers and defensive players, with their team names, position types, etc. Finally these players were exported to CSV using the to_csv method for the Pandas (2010) dataframe to have stored for use in the future or presented to the rest of the NFL team.

#### Beyond the original specifications:
As mentioned above, web scraping was performed to retrieve the team logos. This was done by web scraping content from the CFB (2020) github repository and saving each logo with the team name as a PNG in a folder in the Python project folder. To do this each link in the repository was indexed in a for loop and then saved down as the name of the team for each logo. This code was shown earlier. To then access these paths, a function was created to grab the path for the and convert it to a Python object (see code below):
```python
def getImage(path): 
    return OffsetImage(plt.imread(path), zoom=.2)
```
These were then matched to the defensive targets and displayed in the chart as shown earlier. See code lines 229 - 298 for more detail.

#### Results:

As discussed previously, the final result of this analysis was a csv with the top prospects for the defensive players, kickers, and quarterbacks. The significance of these results are that scouts working for an NFL team could take this list of players to be utilized during the 2021 NFL Draft. 24 individual players were identified as possible targets based on strong performance on relevant statistics for their position group. These statistics were also displayed graphically to assess performance. This was the goal of the analysis. These players can be prioritized during the draft and chosen if still available when the team is making selections. This process can also be repeated for future years with Pro Football Focus data from future college seasons to continue identifying players.

Additionally, as part of the analysis intermediate results were obtained and plotted. The defensive plots are shown below.

![Defensive Bar Chart ](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Defensive%20Summary.png?raw=true)

First the bar plot shows all the accumulated statistics for the defensive players and how the players compare to each other. These statistics were Interceptions, Tackles, Forced Fumbles, Fumble Recoveries, Defensive Touchdowns, Sacks and Missed Tackles. As shown, certain players have more tackles than others, but also have more missed tackle based on the attempts. It is easy to tell which position group is which: the players with more tackles are the linebackers, the players with more sacks are the pass rushers and the players with the least amount of tackles but more interceptions are defensive backs. This chart is useful to show the differences in how each position is different, but also how each position plays an important part in the overall defense. This plot can be used to compare players of similar position groups in the future and see performance for key target players. Based on the current chart, Nick Jackson with the least missed tackles may be the strongest pick for linebacker, while Jeffrey Johnson may be the best target for pass rushers with the most sacks.

![Logos Chart](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Logos%20Sacks%20vs.%20Tackles.png?raw=true)

The second plot, plots the key pass rusher statistics of sacks against tackles. Tackles are on the x-axis and sacks are on the y axis. This chart allows similar visualization of performance as the bar plot, but in a different way and more focused. Ideally, a player would have a high number of tackles and sacks and be in the top right corner of the plot. This is not the case with these players, but as mentioned before, Nick Jackson is a strong prospect (and plays for UVA!) with five sacks and 76 tackles.

For the kickers, their performance was plotted with all their field goal attempts based on distance. As shown below, red represents a miss, and green a make. While Will Reichard did not miss any field goals, Jose Borregales attempted the most from further distance. This could be done for all kickers and compared, but in the case of these three prospects the case should be made that Jose Borregales is the strongest kicker. All three, though, have strong performances based on the analysis.

![Miami Kicker](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Miami%20Kicks.png?raw=true)
![Alabama Kicker](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Alabama%20Kicks.png?raw=true)
![Auburn Kicker](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Auburn%20Kicks.png?raw=truehttps://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Logos%20Sacks%20vs.%20Tackles.png?raw=true)  


For the quarterbacks, it is useful to visualize performance based on all their attempts. To do this, the distance from the left sideline is plotted on the x-axis and distance from the line of scrimmage was plotted on the y axis. As shown, Zach Wilson attempted less difficult passes with most of his throws to the sideline and short distance. Trevor Lawrence and Mac Jones, on the other hand, had more throws of long distance and in the middle of the field. Justin fields only played a few games this season so his plot shows less attempts. While all quarterbacks played in different offensives, it is pretty clear that Trevor Lawrence is a strong prospect with many completions with varying distances and areas of the field.

![Zach Wislon](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Zach%20Wilson%20Passes.png?raw=true)
![Mac Jones](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Mac%20Jones%20Passes.png?raw=true)
![Trevor Lawrence](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Trevor%20Lawrence%20Passes.png?raw=true) 
![Justin Fields](https://github.com/andrejerkelens/prospecting_cfb/blob/main/cfb_project_folder/Plots/Justin%20Fields%20Passes.png?raw=true)  

#### Testing: 
Due to the exploratory nature of this analysis, there were not many unit tests needed. A few were done to ensure accuracy of the data. The first unit test performed was to check that the Pro Football Focus data was imported correctly. The data was checked to ensure not corrupted and that there were the correct numbers of rows. There were 101,731 rows in this dataset and it was to be confirmed that the dataframe has the same amount. This was confirmed with no failures. See below for the code:

```python
class Testcsv(unittest.TestCase):
    def test1(self):
        self.assertEqual(df.shape[0], 101731)
        
if __name__ == '__main__':
   unittest.main()
```
Secondarily, another unit test was run to check if the plots were being added correctly. The test checks to see if the number of plots is more than before after a plot is added. This is also confirmed. See below for the code:

```python
num_figures_after = plt.gcf().number

class TestPlots(unittest.TestCase):
    def test2(self):
        self.assertGreater(num_figures_after, num_figures_before)

#finalize unit test
if __name__ == '__main__':
   unittest.main()
```

#### Conclusions:
As discussed, this program and analysis examined the over 100 thousand plays occurring during the 2020 College Football season and identified 24 individual players as possible targets in the 2021 NFL Draft. This was developed through datamining and visualization using multiple different libraries available with Python and utilized data available through Pro Football Focus. The players were identified based on strong performance on relevant statistics for their position group and these statistics were displayed graphically to assess performance. This process can be repeated and utilized for future NFL drafts by scouts and teams who take an analytic based approach. By datamining the actual plays, the scouts may have an advantage rather than just viewing production. Additionally, visualizing the statistics brings more color and the players to life when deciding whether they are the appropriate player to be drafted to the team.

Some additional improvements to the program could include automating this code into a function so that players are identified and visualizations are displayed. This may be advantageous over actually manually datamining, although some of this will still be necessary to weed out players that may or may not be prime targets. Additionally, it may be useful to find a way to map in the age of the players to help weed out players that are not eligible for the draft (i.e. only 2 college seasons).

If more time was available some additions to the program would be adding in user input to type a players name and, based on their position group, display a relevant plot. Going even further, building this code into an app that allows a user to input a players name and pick the charts to display based with a drop down seems advantageous and something that could be utilized by all teams looking to draft players and evaluate performance.

##### Sources:
1. https://en.wikipedia.org/wiki/2020_NCAA_Division_I_FBS_football_season#Membership_changes
2. https://raw.githubusercontent.com/spfleming/CFB/master/logos.csv
3. McKinney, W., & others. (2010). Data structures for statistical computing in python. In Proceedings of the 9th Python in Science Conference (Vol. 445, pp. 51–56).
4. J.D. Hunter, "Matplotlib: A 2D Graphics Environment", Computing in Science & Engineering, vol. 9, no. 3, pp. 90-95, 2007.
