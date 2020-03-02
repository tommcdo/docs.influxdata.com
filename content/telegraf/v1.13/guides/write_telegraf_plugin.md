---
title: Using the HTTP input plugin with Citi Bike data
description:
menu:
  telegraf_1_13:
    name: Using the HTTP plugin
    weight: 30
    parent: Guides
---
Maybe all your data comes from sources that already have existing Telegraf plugins, but I’m guessing that’s not the case. I’ve made extensive use of the http Telegraf plugin so far, and I’m working on a project that uses the MQTT plugin, but I have been thinking about writing my own plugin as well.

As you’re all aware, I make pretty extensive use of the Particle.io Photon around here—mostly because I have a bunch handy, but look for some work with other IoT devices to come in the near future! I’ve mostly hacked my way around the Particle Cloud so that I could insert data directly from the Photon device into InfluxDB directly, but for this exercise, I thought I’d build a Telegraf plugin that can interact directly with the Particle Cloud’s Webhooks architecture.

First we’ll look at the existing Webhooks plugins to get an idea of how they’re built and structured, and then we’ll modify one to create our own Particle.io Webhooks Telegraf Plugin.

### The Particle.io Webhooks
Let’s look at the Particle.io Webhooks first, so we’ll know how to build our Telegraf plugin later. As outlined in their documentation:

SafariScreenSnapz028

Pretty straightforward, and what we will be building will be one of the ‘Web Services’ shown above. So, off we go to our Particle.io Console to create a new Integration, and choose ‘Webhook’ as the integration type. I’ll be adding this web hook to my existing IoT demo project, so I’ll set the Event Name to ‘temperature’ and for the URL, I’ll use the URL of my Telegraf instance in the cloud.

One thing that is probably unique to Particle is that publishing floats is not supported. To get around that, I just created a macro:

```
#define temp(x) String(x)Copy
```

So that I could turn the float into a String. I then added the publish call to my program loop:

```
Particle.publish("temperature", temp(fTemp), PRIVATE);Copy
```

And I was ready to go. I can then see in the Particle Web Console that ‘temperature’ events are being published every second. So now it’s on to making a Telegraf plugin to pull those events into InfluxDB!

### The Telegraf Webhooks Plugins

On GitHub there is a whole section of nothing but Webhooks plugins for services like GitHub itself, Rollbar, etc. So we’ll start there and look at the Rollbar plugin. I forked the Telegraf repository into my own account so that I could change it to add a Particle plugin. (Note: I had to fork all of Telegraf to do this, but that’s not a huge deal.) And now that I have it all here, it’s time to dive in.

First, I simply duplicated the entire Rollbar directory and renamed it ‘particle’ and then started making the changes I wanted. The two files I need to change are ‘rollbar_webhooks_events.go’ and ‘rollbar_webhooks.go’ but the first order of business is to change everything that says ‘Rollbar’ to say ‘particle’. A global replace inside all the files for ‘Rollbar’ to ‘particle’ and ‘Rollbar’ to ‘Particle’ is also in order.

Next, I took out a bunch of stuff from ‘rollbar_webhooks.go’ that I didn’t need. Specifically:

```
func NewEvent(dummyEvent *dummyEvent, data []byte) (Event, error) {
 switch dummyEvent.EventName {
 case "new_item":
 return generateEvent(&NewItem{}, data)
 case "deploy":
 return generateEvent(&Deploy{}, data)
 default:
 return nil, errors.New("Not implemented type: " + dummyEvent.EventName)
 }
}
```

Since I’m only generating one type of event here, for simplicity’s sake. Here’s my entire ‘particle_webhooks.go’ file:

```
package particle
import (
   "encoding/json"
   "github.com/gorilla/mux"
   "github.com/influxdata/telegraf"
   "io/ioutil"
   "log"
   "net/http"
   "time"
)
type ParticleWebhook struct {
   Path string
   acc telegraf.Accumulator
}
func (rb *ParticleWebhook) Register(router *mux.Router, acc telegraf.Accumulator) {
   router.HandleFunc(rb.Path, rb.eventHandler).Methods("POST")
   log.Printf("I! Started the webhooks_particle on %s\n", rb.Path)
   rb.acc = acc
}
func (rb *ParticleWebhook) eventHandler(w http.ResponseWriter, r *http.Request) {
   defer r.Body.Close()
   data, err := ioutil.ReadAll(r.Body)
   if err != nil {
       w.WriteHeader(http.StatusBadRequest)
       return
   }
   var event ParticleData
   err = json.Unmarshal(data, &event)
   if err != nil {
       w.WriteHeader(http.StatusBadRequest)
       return
   }
   fields, err := event.Fields()
   if err != nil {
       w.WriteHeader(http.StatusBadRequest)
       return
   }
   rb.acc.AddFields("particle_webhooks", fields, event.Tags(), time.Now())
   w.WriteHeader(http.StatusOK)
}
```

Fairly straightforward, and even as a non-Go programmer, I was able to figure out what was going on here and get it working. The trickier bit, for me as a non-Go programmer (yet!) was the next part in ‘particle_webhooks_events.go’ so I had some help.

Basically, I needed to have some structures defined that would correspond to the fields in the incoming json from Particle — more on the Particle json later. So here’s what we ended up with:

package particle

import (
    "strconv"
)

type Event interface {
    Tags() map[string]string
    Fields() map[string]interface{}
}

type ParticleData struct {
    Event string `json:"name"`
    Data  string `json:"data"`
    coreid string `json:"coreid"`
}

func (pd *ParticleData) Tags() map[string]string {
    return map[string]string{
	"id": pd.coreid,
    }
}

func (pd *ParticleData) Fields() (map[string]interface{}, error) {
    f, err := strconv.ParseFloat(pd.Data, 64)
    if err != nil {
	return nil, err
    }
    return map[string]interface{}{
	"temp_f": f,
    }, nil
}Copy
Let’s go through that a little, in case it’s as confusing to you as it was to me. First is the ParticleData struct which lays out the fields that I’m interested in from the incoming json object. That was clear enough to me. The two mapping functions were less clear. Basically, they map the “coreid” field in the ParticleData object to the ‘id’ tag in my database, and then map the ‘Data’ field to the ‘temp_f’ field in my database. There’s a little hitch in there wherein I had to convert the Data field to a double value from a string because the Particle device can only publish INTs and Strings.

Once that was done, I had to go back to the top-level of the Telegraf source tree and build Telegraf using ‘make’, and then deploy the resulting binary to my InfluxDB server. (Hint: If, like me, you’re developing on a Mac and deploying on a Linux box, set the two environment variables “GOOS=linux” and “GOARCH=amd64” and you’ll get a cross-complied version. I love that!)

Adjusting the Particle json
Now it’s time to talk a little bit about the particle device itself, and the whole json business from Particle. We already changed the output from the Particle Device itself to publish the temperature data, so that works fine. Next, we have to go into the Particle Console and define the Webhook for the published event.

SafariScreenSnapz029

You can see that I’ve already defined it, but I’ll walk you through the definition as well. It’s important to remember that the Particle Webhook we built earlier will run on http://<yourhost>:1619/particle, so in your Webhook definition you’ll put that as the URL. Now, here’s the tricky bit, and it took me a while to debug this, so follow along. The default of the Webhook is to send your data as webform-encoded data, and that’s not what you want. But you won’t know it’s webform-encoded if you go and look at the events on the console’s event viewer, because they are listed as:

{"data":"78.763999","ttl":60,"published_at":"2017-09-28T18:22:00.701Z","coreid”:"xxxxxxxxx","userid”:"yyyyyyyyy","version":10,"public":false,"productID":5343,"name":"temperature"}Copy
which looks suspiciously like json data. But that’s not what is actually sent! So while you’re defining your Webhook, look under the ‘Advanced Settings’ and define something, anything in the json field:

SafariScreenSnapz030

That will force the data to be sent to your endpoint as json. Why it works that way I have no idea, but it does.

And now, with my custom-built version of Telegraf, and my newly defined Webhook from Particle, my device is sending data to the InfluxDB instance via my Particle Telegraf plugin.

Final Thoughts
This was a surprisingly easy task, other than the debugging of weird behavior thanks to json shenanigans. With some help from a colleague on the GO side, and then a bit of further work, writing the Telegraf plugin took me about a day or so. Ultimately I’d like to expand and generalize it a bit so that it’s not so specific to my needs—have it parse the readings that it is sent, and put them into the database based on other fields in the json—but for now, I have a Particle Telegraf plugin that logs the data I want.
