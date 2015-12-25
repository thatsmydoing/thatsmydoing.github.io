<!--
.. title: Cloudflare Shenanigans
.. slug: cloudflare-shenanigans
.. date: 2015-12-25 14:13:26 UTC+08:00
.. tags: sysadmin, cloudflare
.. category:
.. link:
.. description:
.. type: text
-->

An old client of ours managed to convince a telco to zero-rate the data for their app. In order to whitelist it though, we needed to use plain HTTP for domain whitelisting. For HTTPS, they can only whitelist by IP address. Like any good developer, we were using HTTPS. Also, like any good developer, we put our server behind Cloudflare.

Now the problem is that Cloudflare can put you behind [any IP they own](https://www.cloudflare.com/ips/), which is a huge range. There's no guarantee that the IP we have now is going to be the same later on. So we did the reasonable thing and asked them to whitelist all of the Cloudflare IPs. And the telco agreed! We were in total disbelief when that happened. But hey, if life gives you free internet, you take it.

We never actually empirically tested whether other sites hosted on Cloudflare were also actually zero-rated. But I like to think that we saved a lot of people on their data costs from browsing Reddit and 4chan. But alas, good things must come to an end.

A few months after we started beta testing the app, Cloudflare added more IPs to their range. Unfortunately, our server got moved to those new IPs which were not whitelisted yet. Apparently, the telco whitelisting process was incredibly convoluted and time consuming. Our client didn't want to bother asking them to whitelist more IPs. We also tried asking Cloudflare to move us back to the original IP range, but they could only do that if we were in their enterprise tier. We couldn't really afford that, so we looked for other options.

Since Cloudflare was essentially just a giant reverse proxy, theoretically there should be no distinction between one IP address from another. The specific IP we get is probably just for load balancing. So we tried accessing the IPs in the range directly and just setting the Host header and it worked! But we get SSL errors because the IP itself doesn't have its own certificate.

After more testing, we figured out that you could actually use any Cloudflare backed domain so long as we properly set the Host header. We just needed to find one still in the old range. Coincidentally, 4chan.org was. Which led to this wonderful commit

    :::diff
    commit 123456789abcdef
    Author: ~~~~~~
    Date:   ~~~~~~

        4chan hack

    diff --git a/src/com/client/common/Util.java b/src/com/client/common/Util.java
    --- a/src/com/client/common/Util.java
    +++ b/src/com/client/common/Util.java
    @@ -210,7 +210,8 @@ public class Util {
            }

            public static String getServerAddress(Context context) {
    -               String address = "https://backend.client.com";
    +               // String address = "https://backend.client.com";
    +               String address = "https://4chan.org";
                    if(!isDebug(context)) return address;
                    try {
    diff --git a/src/com/client/common/logging/APIClient.java b/src/com/client/common/logging/APIClient.java
    --- a/src/com/client/common/logging/APIClient.java
    +++ b/src/com/client/common/logging/APIClient.java
    @@ -101,6 +101,7 @@ public class APIClient {
            private HttpResponse postInternal(String url, List<NameValuePair> data, boolean forRegistration) throws ClientProtocolException, IOException {
                    HttpPost request = new HttpPost(Util.getServerAddress(mContext)+"/api/"+url);
                    request.setHeader("X-API-VERSION", apiVersion);
    +               request.setHeader("Host", "backend.client.com");

                    if(data == null) {
                            data = new ArrayList<NameValuePair>();

Eventually, we did decide to just abandon Cloudflare for the server. We probably weren't going to be the target of a DDOS or anything. This also allowed us to do more secure things like pinning the server certificate in the application itself. Clearly, this is what we should have just done in the first place, but at the time we just wanted a stopgap solution.

I just still find it funny we were making people's phones go to 4chan.org everyday for more than a year.
