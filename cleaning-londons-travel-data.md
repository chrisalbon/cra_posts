# Cleaning London's Dirty Public Transport Data
- Chris R. Albon
- ChrisAlbon
- 2013/11/20
- Data
- draft

The good folks over at the [Open Knowledge Foundation](http://okfnlabs.org/) have launched [a new mini-project](http://okfnlabs.org/bad-data/) around highlighting examples of "bad data." The truth is that I cannot remember a time I've loaded a dataset that didn't require at least some cleaning; cleaning data comes with the job. With that in mind, one thing I have been wanting to do for a long time is demonstrate an example of what "cleaning dirty data" actually means. OFK's Bad Data project gave me the perfect chance (that and a long plane flight) to do just that.

In the long post below, I take [one dataset OFK](http://okfnlabs.org/bad-data/ex/tfl-passenger-numbers/) has highlighted as particularly bad, and explain in plain english each step as I clean it up. I have also included the R code (with extensive comments). This is not a tutorial for data cleaning, rather it is more of a data cleaning demo, with the goal of hopefully at least partially demystying the one aspect of data science.

The data that I am using is transportation data from the city of London. When describing why the data was bad, OFK mentioned these problems:

- First column lacks a heading
- Dates are in an unrecognized format
- Percent symbols in the cells with percents
- Empty rows and columns at the margins of the dataset

The first thing I always do when I get a dataset is pretty simple, I take a look at it. In the code below, I download (e.g. "read") the data directly from the UK government website, then I view the "head" of the dataset, meaning the top few rows, and finally I examine at the structure of the data.

	> # Cleaing Up OKF's Dirty London Transport Data
	> 
	> # load the csv file into R
	> lon.df <- read.csv("http://data.london.gov.uk/datafiles/transport/tfl_passengers.csv", header=TRUE)
	> 
	> # view the top few lines of the dataset
	> head(lon.df)
	> [clipped]
	> 
	> # look at the structure of the dataset, take careful note of the class of the columns
	> str(lon.df)
	> [clipped]

For space reasons, I don't show the output from these commands. However, from this initial assessment I can see the main problems:

1. Variable names (i.e. column names) are long and ungainly, which will make working with them annoying.
2. The first column, which contains date information, does not have a name. 
3. The first column, as OFK points out, has a strange data format "2006/2007 - 1", with the "1" counting up to 13 before starting a new year pair.
4. Many of the columns are being considered by R to be "factors", that is, categories, rather than numbers. This is because the percentage sign is included in many of the cells that denote a percentage change.
5. Extra empty rows and columns.

Okay, now that we know what we are dealing with, let's get to work. 

### Remove Extra Rows

Eyeballing the dataset, I can see that there are 96 observations with real data. However, trailing afterwards is a number of empty rows that serve to purpose. So, let's get rid of them by only keeping the first 96 rows.

	> # remove the empty rows at the bottom
	> lon.df <- lon.df[1:96,]

### Add A Name To The First Variable

In the code below, I add the name to the first variable that didn't have a name. Simple and easy.

	# rename the unnamed first column "date.raw"
	names(lon.df)[1]<-"date.raw"


### Rename Each Variable And Fix Them Up, Part 1

I hate long variable names, it clutters up my code and can increase the change of human error (i.e. me mistyping something), so let's rename the variables to something that is cleaner. Furthermore, I am not a fan of "thousands" or "millions" being units. I prefer to be given the full number and then I can format it as I wish. In the code below, I rename a variable called "Millions.LU.journeys.adjusted.for.odd.days" to "lu.trips" (i.e. London Underground Trips). I also convert the value to it's raw state by multlying it by 1,000,000.


	> # rename "Millions.LU.journeys.adjusted.for.odd.days" column "lu.trips"
	> names(lon.df)[2] <- "lu.trips"
	> 
	> # convert lu.trips units
	> lon.df$lu.trips <- lon.df$lu.trips * 1000000

I repeat this same process for third through seventh variables, which are all structured the same way.

	> # rename "Millions.Bus.journeys.adjusted.for.odd.days" column "bus.trips"
	> names(lon.df)[3] <- "bus.trips"
	> 
	> # convert bus.trip units
	> lon.df$bus.trips <- lon.df$bus.trips * 1000000
	> 
	> # rename "Millions.bus.plus.underground.journeys.adjusted.for.odd.days...new.measure" column "all.trips"
	> names(lon.df)[4] <- "all.trips"
	> 
	> # convert all.trip units
	> lon.df$all.trips <- lon.df$all.trips * 1000000
	> 
	> # rename "LU.Average" column "lu.avg"
	> names(lon.df)[5] <- "lu.avg"
	> 
	> # convert lu.avg units
	> lon.df$lu.avg <- lon.df$lu.avg * 1000000
	> 
	> # rename "Bus.Average" column "bus.avg"
	> names(lon.df)[6] <- "bus.avg"
	> 
	> # convert lu.avg units
	> lon.df$bus.avg <- lon.df$bus.avg * 1000000
	> 
	> # rename "LU.plus.bus.average" column "all.avg"
	> names(lon.df)[7] <- "all.avg"
	> 
	> # convert lu.avg units
	> lon.df$all.avg <- lon.df$all.avg * 1000000

### Rename Each Variable And Fix Them Up, Part 2

The 8th through 13th variables are all structured the name way: they are percentage changes in transportation rates.

To clean them up, I rename them to something simple, in this variable's case "lu.change" for the change in the London Underground's ridership totals. Then I convert the variable into a string (i.e. into text), remove the percent symbol, convert them into numeric (i.e. a number), and divide them value by 100. The reason for the division is simply prefernce, since I am removing the percent symbol, I don't want to confuse people by keeping it as a percentage.

	> # rename "LU.Growth" column "lu.change"
	> names(lon.df)[8] <- "lu.change"
	> 
	> # convert lu.change to a character string
	> lon.df$lu.change <- as.character(lon.df$lu.change)
	> 
	> # replace any "%" with nothing
	> lon.df$lu.change <- str_replace(lon.df$lu.change, "%", "")
	> 
	> # convert lu.change to numeric and change to a decimal
	> lon.df$lu.change <- as.numeric(lon.df$lu.change)/100

Now I repeat those steps for the 9th through 13th variables

	# rename "Bus.Growth" column "bus.change"
	names(lon.df)[9] <- "bus.change"

	# convert bus.change to a character string
	lon.df$bus.change <- as.character(lon.df$bus.change)

	# replace any "%" with nothing
	lon.df$bus.change <- str_replace(lon.df$bus.change, "%", "")

	# convert bus.change to numeric and change to a decimal
	lon.df$bus.change <- as.numeric(lon.df$bus.change)/100

	# rename "LU.plus.bus.growth" column "all.change"
	names(lon.df)[10] <- "all.change"

	# convert all.change to a character string
	lon.df$all.change <- as.character(lon.df$all.change)

	# replace any "%" with nothing
	lon.df$all.change <- str_replace(lon.df$all.change, "%", "")

	# convert bus.change to numeric and change to a decimal
	lon.df$all.change <- as.numeric(lon.df$all.change)/100

	# rename "LU.moving.average.annual.growth" column "lu.avg.change"
	names(lon.df)[11] <- "lu.avg.change"

	# convert lu.avg.change to a character string
	lon.df$lu.avg.change <- as.character(lon.df$lu.avg.change)

	# replace any "%" with nothing
	lon.df$lu.avg.change <- str_replace(lon.df$lu.avg.change, "%", "")

	# convert bus.change to numeric and change to a decimal
	lon.df$lu.avg.change <- as.numeric(lon.df$lu.avg.change)/100

	# rename "Bus.moving.average.annual.growth" column "bus.avg.change"
	names(lon.df)[12] <- "bus.avg.change"

	# convert bus.avg.change to a character string
	lon.df$bus.avg.change <- as.character(lon.df$bus.avg.change)

	# replace any "%" with nothing
	lon.df$bus.avg.change <- str_replace(lon.df$bus.avg.change, "%", "")

	# convert bus.change to numeric and change to a decimal
	lon.df$bus.avg.change <- as.numeric(lon.df$bus.avg.change)/100

	# rename "LU.plus.busmoving.average.annual.growth" column "all.avg.change"
	names(lon.df)[13] <- "all.avg.change"

	# convert bus.change to a character string
	lon.df$all.avg.change <- as.character(lon.df$all.avg.change)

	# replace any "%" with nothing
	lon.df$all.avg.change <- str_replace(lon.df$all.avg.change, "%", "")

	# convert all.avg.change to numeric and change to a decimal
	lon.df$all.avg.change <- as.numeric(lon.df$all.avg.change)/100

### Drop Extra Columns

The original dataset came with eight extra columns. They are the 14th through the 21st column. We can drop those quickly with a single line.

	# drop the empty columns at the end of the data frame
	lon.df <- lon.df[,c(-14, -15, -16, -17, -18, -19, -20, -21)]

### Fixing The Time Variable

Now for the hard part. The problem with the time variable, as OKF points out, is that it doesn't make sense. To fix it, let's first take a look at some of it:

	2006/2007 - 1 
	2006/2007 - 2 
	2006/2007 - 3 
	2006/2007 - 4 
	2006/2007 - 5 
	2006/2007 - 6 
	2006/2007 - 7 
	2006/2007 - 8 
	2006/2007 - 9 
	2006/2007 - 10
	2006/2007 - 11 
	2006/2007 - 12 
	2006/2007 - 13 
	2007/2008 - 1 
	2007/2008 - 2 
	2007/2008 - 3 

That is a very strange format. Every row contains two years, a dash, then a number. When that last number reaches 13, it resets to 1 and years increase by 1. Cleaning this will require some creativity.

The number after the digit goes up to 13, so it isn't denoting the month. However, 52 divided 13 is 4! Aha! So that number denotes 4 week blocks!

The years are a little more confusing, since they include two years, it is probably safe to assume that the "year" period they denote overlaps the calendar year, just like how "academic calendars" overlap the calendar year (e.g. "the 2013/2014 academic year").

So, taking the first row, "2006/2007 - 1", we know it refers to a four week block of time that occured in 2006. But when exactly? Since we don't have additional information provided by the dataset, we are going to have to make an assumption. Some quick research online says that fiscal year in the UK starts on April 1st. Great! That'll be our start day. We could be wrong, but there is a good chance we are right.

For the next section, since it is a bit more complex, I'll explain each line.

First, we create an object (i.e. a piece of data) called start.date that denotes April 1st 2006, which we just decided to assume is the start date for the data.

	# create a variable that corresponds to the start of TFL's year
	start.date <- as.Date("2006-04-01")

Second, we create an object that is some date in the distant future (doesn't really matter what)

	# create some event far into the future
	end.date <- as.Date("2020-01-01")

Third, we tell R to write out a sequence of dates, starting at *start.date*, ending at our *end.date*, with the dates spaced 4 weeks apart. We call this variable "date.guess" to express that it isn't verified.

	# create an element for every year between two dates
	date.guess <- seq(start.date, end.date, by = "4 weeks")

Fourth, we take the first 96 of these dates (because there are 96 rows in the dataset)

	# take the first 96 dates (since there are 96 observations in lon.df)
	date.guess <- date.guess[1:96]

Finally, we attach (bind) the column to the dataset

	# add time.period.possible to the lon.df data frame
	lon.df <- cbind(date.guess, lon.df)

### And we're done!

We are left with is nice and clean dataset, with real dates attached to the observations, short informative variable names, and correctly formatted values.

If you are interested in trying this out for yourself, the full R code for this post is avaliable on Github.

### Note: What we didn't do

1. I did not create a codebook for the dataset. Codebooks are the "user's manual" of datasets, typically explaining how the data was collected, how each variable is defined, how it was created, and other things. It is exceptionally useful, but for this example I decided against it out of laziness.

2. I didn't write the most eloquent code. My code is probably twice as long as it needs to be, and I could have combined many of the lines to shorten it up. However, I wanted to explain each step, and this becomes harder to do if a single line of code does many things.

3. My solution to the date problem was pretty good (assuming my guess about the fiscal year start date was accurate), however it is still is not entirely accurate. What do I mean? Well, for example, "2006/2007 - 1" starts of April 1st, but the start date drifts each year, so that the start date for "2013/2014 - 1" is March 23rd. The reason is not that interesting, but needless to say that if I needed to be very precise about the date, I'd redo it to fix that problem. But for now it is okay, particularly because we don't know if April 1st is even the correct date to being with. And, well, frankly my flight is about to land.