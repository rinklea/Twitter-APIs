import requests
import json
import time
from TwitterAPI import TwitterAPI
import tweepy as tw

start_time = time.time()

CONSUMER_KEY = '<consumer key>'
CONSUMER_SECRET = '<consumer secret>'
ACCESS_TOKEN = '<access token>'
ACCESS_TOKEN_SECRET = '<access token secret>'


apis = TwitterAPI(CONSUMER_KEY
                  CONSUMER_SECRET,
                  ACCESS_TOKEN,
                  ACCESS_TOKEN_SECRET

# Authenticate to Twitter

auth = tweepy.OAuthHandler(CONSUMER_KEY, CONSUMER_SECRET)
auth.set_access_token(ACCESS_TOKEN, ACCESS_TOKEN_SECRET)
api = tw.API(auth, wait_on_rate_limit=True)

params = (
    ('query', 'nyc'),
    ('tweet.fields', 'source'),
    ('expansions', 'author_id'),
    ('user.fields', 'username')

)
headers = {
    'Authorization': '<AUTHORIZATION TOKEN>'
}

info = []
user = []


def collectTweets1():
    search_words = ["<search words>"]
    date_since = "<date>"

    for words in search_words:
        # Collect tweets
        tweets = tw.Cursor(api.search,
                           q=words,
                           lang="en",
                           since=date_since).items(1)

        for tweet in tweets:
            info.append(tweet.user.id)
            user.append(tweet.user.screen_name)


def collectTweets2():
    for tweets in range(1):
        response = requests.get('https://api.twitter.com/2/tweets/search/recent', headers=headers, params=params)
        res_dict = json.loads(response.text)
        for j in res_dict['data']:
            if j["source"] == "<tweet source":
                if not j['author_id'] in info:
                    info.append(j['author_id'])
                    url = "https://api.twitter.com/2/users/" + str(j['author_id'])
                    responses = requests.get(url, headers=headers)
                    user_dict = json.loads(responses.text)
                    user.append(user_dict['data']['username'])


def dm():
    for i in info:
        user_id = i
        message_text = "hi"
        event = {
            "event": {
                "type": "message_create",
                "message_create": {
                    "target": {
                        "recipient_id": user_id
                    },
                    "message_data": {
                        "text": message_text
                    }
                }
            }
        }
        r = apis.request('direct_messages/events/new', json.dumps(event))
        print(r.text)


def mention():
    for each_user in user:
        api.update_status("welcome  @" + each_user)


collectTweets1()
collectTweets2()
dm()
mention()


print("--- %s seconds ---" % (time.time() - start_time))
