# podcast-hosts
A JSON list of podcast hosts, and the patterns they use in audio URLs.

If you have the URL of a podcast's audio file, this JSON file will help you work out who the host is; and abilities that host may have that may impact listener privacy.

This data is [synched to podcast-privacy.com](https://github.com/fancysoups/podcast-privacy/issues/2#issuecomment-720155022), which offers an API.

## Images

Each podcast host has an image, in optimised .png format. The filename is programmatically calculated from the podcast host's name, in lower case, with non-alphanumeric characters encoded as a "-". Here's our ugly code:

`$imageFilename=str_replace(' ','-',str_replace('.','-',strtolower($host['hostname'])));`

## Contributing to the list

For now, the simplest way is to add to the file at `src/hosts.json`. Each podcast host may have multiple entries, but the URL patterns should be unique.

Each entry _must_ contain the following properties:

* `pattern`: a unique string to spot within the audio URL (more accurately, within the domain portion of the URL)

* `hostname`: a humanly-readable name of the podcast that this is hosted with

* `retailer`: a boolean showing whether podcasters can buy hosting on this podcast host (BBC isn't; Libsyn is).

* `iab`: a boolean showing whether a podcast host is able to offer [IAB Certified Compliant statistics](https://iabtechlab.com/compliance-programs/compliant-companies/#podcast). This does not mean that all statistics from this host are certified compliant.

* `abilities_stats`: a boolean showing whether this podcast host uses its logs to provide stats

* `abilities_tracking`: a boolean showing whether this podcast host uses its logs to provide tracking and attribution

* `abilities_dynamicaudio`: a boolean showing whether this podcast host is capable of dynamic content insertion, used for advertising or content.

* `hosturl`: a website link (escaped) that links to the homepage of the podcast host.

Each entry _can_ contain the following properties:

* `rss-pattern`: an ADDITIONAL pattern for the RSS URL, to help with cases like LibsynPro which uses the same infrastructure for hosting retail and non-retail shows

* `privacyhosturl`: a link to the privacy policy of the podcast host

* `notes`: a freeform text link with details of evidence of 'abilities' claims. Ideally, all podcast hosts that have these indicated will have evidence in this field.

## Code sample

Podnews uses the below to extract a host's name, and privacy details, in podcast pages ([example](https://podnews.net/podcast/1287081706)).

```
$audioUrlArray=parse_url($podcast['audiourl']);
$audioUrlNoQueries=$audioUrlArray['host'].$audioUrlArray['path'];
$stmt = $db->prepare("SELECT * FROM `podcasts-hosts` WHERE (INSTR(:url,pattern) AND INSTR(:rssurl,`rss-pattern`)) OR INSTR(:url,pattern) ORDER BY `rss-pattern` DESC LIMIT 1");   
$stmt->execute(array(':url'=>$audioUrlNoQueries,':rssurl'=>$podcast['feedUrl']));
$host = $stmt->fetch(PDO::FETCH_ASSOC);
```

(You're recommended to remove the query string, as above).

Improvements are welcome. This list won't adequately spot original hosts via Feedburner or similar, but otherwise will catch most podcasts.
