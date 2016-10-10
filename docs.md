# Welcome
A brief overview of the FOMO.vote API that is currently set up to provide polling location wait times for the November 2016 elections in Travis County. We want to work with as many people as possible to collect data about line locations, so that we can, in turn, generate more reliable estimates of wait times.

If you're interested in sending us data from your users, we don't expect you to implement all of these functions. In fact, our own app is likely only going to collect Binary Reports. If you only want to show line estimates, that's okay too! We have one goal and it's to get people out to vote. The more people spreading this information, the better.

Please feel free to report any problems or feedback you have with our API using Github issues linked above.



## Opinionated
We are planning on producing two indicators of polling station line status. The first is a plain count of how many minutes we expect someone to wait, based solely on actual reports from a trailing 30 minute period. This is the "safe" number to use because it will always be roughly correct on aggregate. 

The second is our predictive rating of how long the line is at the current moment, which we indicate on a scale of 0 to 1, where 1 is annoyingly long for the average person. We're fuzzy on what it takes for a polling location to approach 1 because we'll probably be tinkering with the algorithm on voting day. We're opinionated about lines, in the sense that we think people are willing to stand in line for a very short period of time before getting annoyed. 

In other words, it's possible to get back a 7 minute wait time with a 0.803 rating, because we believe that the polling location is currently getting worse and we just don't yet have actual reports. 

# Sending us data
We can improve our estimates if you are willing to engage your users to send in their reports. We accept a variety of inputs on wait times, so feel free to pick and choose the one(s) you find appropriate for your application.

### Our base endpoint
>https&#58;//v1.api.fomo.vote/report/

You'll POST data to the appropriate endpoint. We ask that you include app_name as a POST variable with a static reference to your app. This just lets us know who's who, so if there's an issue we can tweet at you.

If you get a 200 response, you're good to go. If you get anything else, we recommend waiting (asynchronously) 30 seconds or so and trying again.

## Binary Report
This is the simplest one, but surprisingly powerful based on our past efforts during the primaries and SXSW.

We recommend asking users if the line is *long* or *short*. Everyone has their own definition of long and short, but we take that into account when generating our predictive line estimates. Please don't send a binary report if you ask the user something like "was your wait longer or shorter than 5 minutes".

When you send us binary reports, it is not used in the minute-based wait data.

>https&#58;//v1.api.fomo.vote/report/**binary**/


### Attributes
| Name        | Options           | Description  |
|----|----|----|
| report      | long, short | Choose one or the other |
| poll_id      | *integer*       |   *Required* Which poll location this report is about. |
| timestamp      | UNIX timestamp      |   *Optional* If not included, we'll use the request received time. |



## Individual Time in Line Report
A user, or your estimate of a user's, amount of time in minutes spent waiting in line, from the point of first entering the line to being able to enter (i.e. the beginning of voting).

>https&#58;//v1.api.fomo.vote/report/**timeinline**/

### Attributes
| Name        | Options           | Description  |
|----|----|----|
| minutes      | *integer* | How long in minutes |
| reportmethod      | self, estimated, measured | If you ask the user to enter the time based on their memory, indicate self. If you have some wizbang way of estimating it, indicate estimated. If you have the user start a timer or some other more accurate way of measuring, indicate measured. |
| poll_id      | *integer*       |   *Required* Which poll location this report is about. |
| timestamp      | UNIX timestamp      |   *Optional* If not included, we'll use the request received time. |

## Count of people in line
This, combined with known data about the particulars of a polling location (chiefly, number of voting booths), helps us estimate the wait time even if the count is an estimate or guess.

The ```reportmethod``` is not factored in heavily in this case, because we generally assume no one is going to give a super accurate count and our algorithm takes that into account. However, it's nice for us to know.

>https&#58;//v1.api.fomo.vote/report/**peopleinline**/

### Attributes
| Name        | Options           | Description  |
|----|----|----|
| people      | *integer* | How many folks are in line |
| reportmethod      | self, estimated, measured | We take self to mean a guess, estimated if you ask them for a range of people in line, and measured if you feel confident that the user actually spent the time counting. |
| poll_id      | *integer*       |   *Required* Which poll location this report is about. |
| timestamp      | UNIX timestamp      |   *Optional* If not included, we'll use the request received time. |

## Poll entrance/exit rate
The rate at which people enter or exit the polls. Our own implementation of this is likely to be a "game" that prompts people to tap a button each time someone leaves the poll over a 5 minute period. 

>https&#58;//v1.api.fomo.vote/report/**rate**/

### Attributes
| Name        | Options           | Description  |
|----|----|----|
| count      | *integer* | How many people entered *or* exited the polling location |
| period      | *integer* | Time period the count was conducted in seconds |
| direction      | enter, exit | Whether the count was for people coming or going. *Recommended*: Use exit for user generated data sources. See note below. |
| poll_id      | *integer*       |   *Required* Which poll location this report is about. |
| timestamp      | UNIX timestamp      |   *Optional* If not included, we'll use the request received time. |

**Note**: We recommend using exit as the count because it's not uncommon for someone to enter the polling station and have to be redirected to resolve an issue with their registration, identification, or similar. 


# Polling Location Line Data

While the polls are open (even during early voting), every 5 minutes we will update the following endpoint that you can pull to get line data. If we're able to process data faster than every 5 minutes, we may elect to do so. 

> https&#58;//v1.data-api.fomo.vote/lines.json

It's a simple JSON file that will look something similar to:
```
{
  "product": "FOMO.vote Line Indicator - Nov2016 TravisCo Voting",
  "version": 0.1,
  "releaseDate": "2016-09-19T00:00:00.000Z",
  "locations": [
    {
      "poll_id": 12345,
      "name": "Austin City Hall",
      "waitTime": 12,
      "fomoRating": 0.321,
      "confidence": 0.8
    },
    {
      "poll_id": 12343,
      "name": "Carver Library",
      "waitTime": 75,
      "fomoRating": 0.987,
      "confidence": 0.2
    }
  ]
}
```

Please check the version number prior to parsing. We won't increment it (or change the format) once voting has started, but it's possible that we'll update prior to.

### Data returned
| Name        | Value           | Description  |
|----|----|----|
| poll_id      | *integer* | A unique ID of the polling location.  |
| name      | *string* | A friendly name of the polling station. |
| waitTime      | *integer*     |   Amount of time people have spent waiting in line, based on reports. |
| fomoRating      | *integer*     |   Our estimate of the wait at the current time, based on our predictions and estimates. 0 is no wait, 1 is a frustratingly long wait. See **Opinionated** for more info.  |
| confidence      | *integer*      |  How confident we are on the accuracy of this report (including *waitTime* and *fomoRating*). This is largely impacted by the number of reports and time since the last report. 0 is not at all confident, 1 is extremely confident.  |

This will be our first attempt at a confidence rating, so we're not sure how useful it is going to be for end users. Feel free to take it into account, but you should expect lots of 0.5 confidence ratings.

## Usage / Limitations

The endpoint is hosted on Amazon S3, so feel free to **not** cache it and request it as frequently as you'd like on behalf of your users. (If you'd like to get data over time programmatically, please contact @EdIreson so that you don't make our AWS bill outrageous by requesting it every 5 seconds or something like that.)

We ask that you credit FOMO.vote if you show this data. It can be tucked away in a credits page or in the footer. Additionally, we would appreciate it if you let us know ahead of time that you are planning on using this data, just so that we can say Hi!

[Say Hi! >](hi@fomo.vote)


