# Indie Game Recommender
Find refuge from the Indiepocalypse with the Indie Game Recommender

#### About this readme
This readme is still a work in progress. Many more diagrams and editorial polishing will be performed but this should give a good initial overview.  

Thank you, 

Brian 

pixelatedbrian@gmail.com

## What is this project?
This the capstone project of Brian Hardenstein.  I'm currently finishing up a Data Science bootcamp with Galvanize at the Seattle campus. The goal of this project is to demonstrate the skills that I have acquired in the course of this program. Ideally it will demonstrate a good business understanding of the needs of those that the project will serve, ie players and developers. It will also show the combination of skills required to go from a business question, "how do we get more great games discovered by people who will like them?", to an end product that recommends undiscovered games to new players.

In the course of this project the following project pipeline is being implemented:

![Process Pipeline](/imgs/process.png)

## Who are you and why are you making a Indie Game Recommender?
My name is Brian Hardenstein and I am currently transitioning from being a Network Engineer to being a Data Scientist. In my previous job I was a consultant who analyzed a lot of client data and then tried to determine what information and take-aways were most useful and insightful through the lens of business utility and adding value. Over time I came to realize that I enjoyed the analysis and search for insight more than the engineering aspect. Through self studuy I started to learn about statistics and probability and I eventually applied to Galvanize's Data Science 3 month bootcamp in Seattle. This project is the culmination of that program. 

With regards to why an Indie Game Recommender, I've always been a pretty big fan of games. The Indie Wave with some really classic games really got me interested in the Indie scene. (Spelunky, Terraria, The Binding of Isaac, etc) A lot of gamers, myself included, have daydreamed of making their own game. I've actually worked on many little projects of my own, mostly in Unity using C#. 

![Brickbreaking Prototype, Brian Hardenstein 2015](/imgs/brick_proto.gif)
![Airplane Prototype, Brian Hardenstein 2016](/imgs/airplane_proto.gif)

I don't really have dreams of making it big as a game dev but I do think that it's a great creative outlet and that the bar of entry is getting lower and lower. This is great in that it helps democratize games and get many new viewpoints and visions out there that would have never existed otherwise. On the other hand it also means that there is a complete saturation of the gaming environment. A lot of indie games aren't that great and it's a bit hard to find great new ones. (With limited experience and resources it's understandable that some games aren't quite as good.  However, that still doesn't mean that people should buy them, per say.) I figured that a project like this would help some of the struggling game devs who haven't gone viral but have managed to create a fun game to play.

## Why Indie Games?
It depends on the player and what he or she is interested in but there are some common themes on why people like games made by independent developers. Some of those themes include:
- More freedom to be creative and/or experimental
- More artistic freedom
- Indie devs can cater to smaller niches with more unique content
- Frequently supporting a small business/small team/individual

## Why an Indie Game Recommender?
There's a lot of annecdotal evidence of a "Indiepocalypse" that essentially boils down to: 
Due to the sheer quantity of games being released it is very difficult to get noticed even if one develops a quality game. This is even more difficult for Indie Developers because they don't have the resources/skills/etc to evangelize their game effectively in an environment that is saturated with a lot of competing attention.

The goal of this Indie Game Recommender is to help quality games that are relatively unknown get shown to a broader audience. If the recommendations are of high quality then it is a win/win/win scenario. Players gain access to a game that will delight them that they may have never encountered otherwise. Developers who have made a quality product would be more likely to sell games to individuals who have already shown an affinity for games that are similar to theirs.  Finally if the marketplace operates with less friction then this benefits Valve, who operates the Steam platform where the data from this project came from.

## Why just data from Steam?
As an avid gamer I use steam for nearly all of my PC game playing. So it's the first thing that came to mind. There's also a Steam API which helped me gather the data. Also, with my interest in developing an indie game of my own, Steam was always the place that I had planned on publishing any game that I might have made.  

I am aware that there are other options for indie games but I also had to limit my scope to something that was feasible for a capstone project.

## What kind of data is used?
For this project I used a combination of data from the Steam API as well as some rudimentary info scraped from the Steam website.  Having a basic Steam API key no player specific information was gathered except for Steam ID's, which where anonymized after scraping. The specific data gathered was mostly games owned and playtime of the games.  Explicit reviews (good/bad ratings) of games were also scraped for games tagged as "Indie" on steam but thus far not used.

## What can you tell me about the model that is being used for the recommender?

#### What kind of model?
Because it's good practice the implementation that I am using is Apache Spark. The precise model that I am using is ALS (Alternating Least Squares). A stretch goal currently is implenting some user bias and item bias compensation similar to that which was used to great effect in the Netflix Challenge.

#### What are you putting into the model?
For the model I am predicting based upon implicit ratings in the form of playtime_forever (minutes) converted to a log10 scale.

For example if I had played Stardew Valley for 1000 minutes then my implicit rating for Stardew Valley would be "3.0".

After this very basic initial processing a sparse 2d matrix is created of M x N dimensions. M being the number of rows, one row for each user in the sample. N is the number of columns and corresponds to one column for each game. If I was User_1 and Stardew Valley was the first column then:

User | Game 0 | Game 1 | Game 2 | ... | Game N | 
--- | --- | --- | --- | --- | --- 
User 0 | 2.34 | 0 | 0| ... | 4.92
User 1 | 0 | 3.84 | 2.99 | ... | 0
User 2 | 0 | 4.73 | 4.20 | ... | 2.43
... | ... | ... | ... | ... | ...
User M | 3.65 | 2.14 | 0| ... | 4.12

Altogether there are ~130K users in the sample (M) and 8200-8700 indie game app_ids (N).  Altogether this means that there are just under 1.1B points of data. There's ~9M games with playtime.  Therefore the M x N matrix is ~0.8% populated by data.

#### Fitting the model
After fitting the ALS model there are two matrices created. Frequently called the UserFactors and ItemFactors, or U/V. Currently most effort is being focused on the V matrix/ItemFactors.  The dimensions of this matrix is (k, N) where k is the "rank" hyperparameter. 

###### Example using rank of 3:

Item Factors **(V)**:

Rank | Game 0 | Game 1 | Game 2 | ... | Game N | 
--- | --- | --- | --- | --- | --- 
Rank 0 | 2.34 | 0 | 0| ... | 4.92
Rank 1 | 0 | 3.84 | 2.99 | ... | 0
Rank 2 | 0 | 4.73 | 4.20 | ... | 2.43


User Factors **(U)**:

User | Rank 0 | Rank 1 | Rank 2 |
--- | --- | --- | --- | 
User 0 | 2.34 | 0 | 0| 
User 1 | 0 | 3.84 | 2.99 | 
User 2 | 0 | 4.73 | 4.20 |
... | ... | ... | ... | 
User M | 3.65 | 2.14 | 0| 

The main reason that we care about rank is because that is the number of latent features that we are looking for when the model is trained.  The simplest way to think about latent features is sort of derived categories that the ALS algorithm is finding in the data of the initial M x N matrix. One can sort of think about latent features as being things like "FPS", "farming simulator", etc.  However it is important to realize that these latent features represent patterns in the data and not exactly genres or intelligible features for us humans.  

The rank is a very important hyperparameter in this model. If the rank is too low then the latent features won't be granular enough to capture the differences of the games being categorized. The higher the rank the more latent features are available to help predict if a player will enjoy a game but then the risk of "overfitting" to the training data is higher.

#### How is the model evaluated?
For algorithmic and objective validation RSME is being calculated using cross-validation. The ALS model is trained on a train set of the data (~60%) and then scored on a test set of the data (~30%).  10% of the data is being held in reserve for the final evaluation. The final evaluation set probably isn't needed in this case but has been done in an attempt to keep to best practices.

I did some light hyperparameter tuning and was able to determine that a rank of 25 was optimal with regards to minimizing the objective error.  Please see the following diagram:

![RMSE vs Rank](/imgs/rmse_vs_rank.png)

As we can generally only use RMSE to evaluate a particular model I used a sort of "worst case recommendation" in order to set the bar. The bar is at RMSE of ~1.78 and was determined by using the mean of all of the rankings in the original utility matrix.  Relative to this worst case scenario the current model RMSE is ~1.68.

For more of a subjective, but probably more important, evaluation one simply eyeballs the predictions versus the reality relative to a given user. A method was written which pulls out the data for a particular anonymized user from the training set, and the testing set.  Those are then compared to what the model is predicting for that particular user.

![Results Example](/imgs/results_example.png)

#### Subjective evaluation triggered two major rounds of model revisions

The first major revision was going from explicit reviews to implicit reviews.

The second major revision was triggered because the model was frequently recommending games that were still under development. Upon investigating this issue I found that the games being recommended were games still in development. This lead to me hypothesizing that the weird recommendations were coming from games that were under development. This lead to further EDA to investigate if that was true.

![Average Game Play EDA](/imgs/EDA.png)

This helped to identify two key clusters. The red cluster on the lower right are games with very high average playtimes and very low unique player counts. This was handled by implementing a weighting system that penalizes low unique player counts. The weighting penalty is applied to a user rating based on how many unique players the game has.

![Base Weighting](/imgs/weighting_s1.png)

The bright green cluster in the top middle of the diagram are games that had very high unique player counts. That cluster of games was also very frequently recommended. In the case where a gamer might want to find more obscure games an optional weighting was also developed that penalizes games with high player counts. 

![Optional Weighting](/imgs/weighting_s3.png)






