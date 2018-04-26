---
title: "HTTP to Upspin gateway"
date: 2018-04-26T09:00:00+03:00
---

First time I heard about [Upspin](https://upspin.io/) from this video
{{< youtube ENLWEfi0Tkg >}}
and I very like it idea. Who don't know what it is about, can check the site https://upspin.io/ 
From the documentation:

>Upspin provides a global name space to name all your files. Given an Upspin name, a file can be shared securely, copied efficiently without “download” and “upload”, and accessed from anywhere that has a network connection.

and recently I deployed upspin server on my GCP VM
{{< tweet 988325620693258240 >}}

but no one of my friends uses upspin yet and I want to have ability to share some of my public files
I decided to write simple HTTP to Upspin gateway.

and it turned out to be quite simple:

I just need to register a separate user in keyserver, you can find information [here](https://upspin.io/doc/signup.md)

put user keys and config to the server and write this simple server:


{{< highlight go >}}
package main

import (
	"bytes"
	"io/ioutil"
	"log"
	"net/http"

	"upspin.io/client"
	"upspin.io/config"
	"upspin.io/transports"
	"upspin.io/upspin"
)

var c upspin.Client

func main() {
	data, err := ioutil.ReadFile("/home/user/upspin/config")
	if err != nil {
		log.Fatal(err)
	}
	cfg, err := config.InitConfig(bytes.NewReader(data))
	if err != nil {
		log.Fatal(err)
	}

	transports.Init(cfg)
	c = client.New(cfg)
	http.HandleFunc("/", handler)
	log.Fatal(http.ListenAndServe(":80", nil))
}

func handler(w http.ResponseWriter, r *http.Request) {
	path := r.URL.Path[1:]
	if path == "favicon.ico" {
		return
	}
	b, err := c.Get(upspin.PathName(path))
	if err != nil {
		log.Print(err)
	}
	w.Write(b)
}

{{< /highlight >}}

and in pieces:

so, here we just read config and initialize upspin config struct
{{< highlight go >}}
	data, err := ioutil.ReadFile("/home/user/upspin/config")
	if err != nil {
		log.Fatal(err)
	}
	cfg, err := config.InitConfig(bytes.NewReader(data))
	if err != nil {
		log.Fatal(err)
	}
{{< /highlight >}}

then we need to initialize transports and create new upspin client
{{< highlight go >}}
	transports.Init(cfg)
	c = client.New(cfg)
{{< /highlight >}}

and then in the handler, we just call `Get` method with `path` that we get from the URL
{{< highlight go >}}
	b, err := c.Get(upspin.PathName(path))
{{< /highlight >}}


now, if you run this server and append upspin path to the URL you will get your file from upspin(if it's publicly available, read about [access control](https://upspin.io/doc/access_control.md))

My server deploy on the domain [up.aborilov.ru](http://up.aborilov.ru), you can try on it, for example: http://up.aborilov.ru/augie@upspin.io/Images/Augie/smaller.jpg

{{% center %}}
![upspin mascot](http://up.aborilov.ru/augie@upspin.io/Images/Augie/smaller.jpg)
{{% /center %}}
