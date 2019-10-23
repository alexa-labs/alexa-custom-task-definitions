# Bedtime Stories Custom Task Definitions

## VolleyBedtimeStories

* Task Name: VolleyBedtimeStories

This task is designed to be requested from other Alexa skills which would like to allow their customers to hear a Volley Bedtime Story.

The VolleyBedtimeStories task allows a customer to select a genre and pick a bedtime story title from a list of (free) options.

The customer can also opt to subscribe for access to over 50 additional bedtime stories.

The bedtime story will begin playing and the task will be marked complete.

* Input parameter:
  * requesterSkillId (type: string)

* Resulting property when the status is `200`:
  * startedBedtimeStory (type: boolean)
