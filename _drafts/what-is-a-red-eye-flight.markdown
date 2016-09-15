---
title:  "What is a Red-eye Flight?"
description: Colloquials present challenges for computer interpretation, partly due to their contextual usage.
## date: add a date when publishing
---

A lexicon of words and phrases that are non-standard or casual are part of everyday conversation. It is fascinating that different cultures and languages share common expressions that have different direct translations. In English, we can describe something as totally incomprehensible to us by declaring "It's all Greek to me.". In Greek, they say "It's Chinese to me.". Things start to get [geopolitical](http://omniglot.com/language/idioms/incomprehensible.php), maybe in the future this will become another data visusalisation. 

Appreciating these idioms is possible for us, as humans, but difficult for machines. This presents challenges in a variety of Natural Language Processing tasks. Consider Sentiment Analysis, a [great example](https://www.quora.com/Are-slang-and-jargon-taken-into-account-in-sentiment-analysis-If-so-how) is the use of the word 'sick' in a passage of text. If we are trying to determine the sentiment of restaurant reviews, this should probably influence the result towards a negative classification. However, in the context of snowboard reviews, the converse applies.

# Red-eye Flights

Don't ask me [why](https://en.wikipedia.org/wiki/Red_Eye_(2005_American_film)), but I thought a red-eye flight was one which travels between the East and West coast of the U.S. It is actually a colloquialism for a flight that takes place overnight, its origins pay respect to the redness of the eyes that can occur with fatigue. Fair enough, that actually makes sense.

Imagine if we could issue a query to a computer using our own natural language, "Show me all red-eye flights". In the not-so-distant future, this type of technology will [exist](https://www.youtube.com/watch?v=N0mRn1bQyzU). Until then, you, or an attendant at an airline, would find this information using reasoning to qualify flights as red-eye or not. In the rest of the post, we see how this qualification depends on a variety of contextual features and discuss how these could be implemented into a system.

First of all, let's consider all possible flights. These can be visualised as routes between airports. If your interested in the tools used to build this visualisation, there is a tutorial on Tom Noda's [blog](http://www.tnoda.com/blog/2014-04-02). For simplicity, each flight has been assigned a take-off time, an integer in the range $$[0,24]$$. 

![all flights](/assets/images/what-is-a-red-eye-flight/all-flights.png "all flights")

## Night flights

The first way to filter this information is to find flights that have a scheduled take-off time within the night. Note that even this definition is debatable, a more robust constraint would be to find flights with a duration that also ensure they land before the break of morning. To keep things simple, let's use the first idea. Next, we need to check what time this actually is. Of course, even this is not straightforward, our planet's tilted earth of rotation ensures this changes seasonally. In a live system, a record of this seasonal information would need to be maintained. Today (15th September), in London, the sun sets at 19:14. Let's see how the map looks showing only flights that happen after this time.

![night flights](/assets/images/what-is-a-red-eye-flight/night-flights.png "night flights")

## Co-ordinated time

![time zone](https://upload.wikimedia.org/wikipedia/commons/e/e8/Standard_World_Time_Zones.png "time zone")

In the previous example, we assumed that all of the take-off times were local. More likely, the flight information is recorded in a central database which makes use of a Co-ordinated Univeral Time (UTC) take-off times. So, we actually need to adjust our take-off times with the airport's corresponding UTC offset. Wikipedia has offsets for [some airports](https://en.wikipedia.org/wiki/List_of_airports_by_IATA_and_ICAO_code), this is actually significant, as in this format the information is accessible to a system that can scrape the web. However, we had to use a [separate library](https://raw.github.com/hroptatyr/dateutils/tzmaps/iata.tzmap) and [moment-timezone](http://momentjs.com/timezone/) to make this possible. This helps to build a more realistic map of the available red-eye flights.

![utc night flights](/assets/images/what-is-a-red-eye-flight/utc-night-flights.png "utc night flights")

## Location awareness

Finally, we are probably only interested in flights from our current location, London. We encode this information to generate a final map of all red-eye flights, relevant to us. This is just a single flight from London Heathrow (LHR) to Shanghai Pudong (PVG).

![utc lhr night flights](/assets/images/what-is-a-red-eye-flight/lhr-utc-night-flights.png "utc night flights")

## Conclusion

By adding more context to our original definition of red-eye flight, we have helped the system to understand our definition and return more useful results.
