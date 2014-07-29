---
layout: post
title: Automatic Update for CodePlex Projects
permalink: /2011/03/automatic-update-for-codeplex-projects.html
---

I recently implemented a check for updates feature on my [Image Resizer for Windows][1] project, and wanted to share my
experience for the benefit of other CodePlex project coordinators and developers.

When looking for a solution, I came across a few features of CodePlex that looked like they might be interesting, but
most of them turned out to be dead ends.

CodePlex Web Services
=====================
[Using the CodePlex Web Services][2] looked promising. This is a web-based API that allows you to create new releases,
and upload files to a release. Unfortunately though, it does not allow you to retrieve existing releases or download
files from a release. Bummer.

RSS feed for release updates
============================
Brilliant, an RSS feed for releases – exactly what I'm looking for! Right? Wrong. The format of this feed has far too
many shortcomings to be useful for our scenario. First, items get added whenever a release is updated (e.g. you fix a
typo). This create an excessive amount of noise in the feed. Second, there is no good way to get a release's version
number without having it be the display name and then parsing it out using regular expressions (bad for the users who
have to look at it, and for the program because it has to do extra processing). And third, nowhere in this feed does it
tell you the specified status (Alpha, Beta, or Stable) of the release. Nuts.

Feature request it...
=====================
...and wait a few years. I did actually (See [CodePlex via OData][3]), but like most requests, it's just sitting there,
collecting votes, waiting for a PM in shining armor to come rescue it from its server in the cloud.

Well, it was time to think outside of the box. I knew that I needed a file to describe the available updates. It had to
have a release date, version number, release status, and a link to the download page. A feed was defiantly what I
needed. Here is how I wanted it to look:

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<feed xmlns="http://www.w3.org/2005/Atom">
  <title type="text">Image Resizer for Windows</title>
  <id>http://imageresizer.codeplex.com/releases</id>
  <updated>2011-02-28T00:00:00Z</updated>
  <author>
    <name>Brice Lambson</name>
  </author>
  <entry>
    <id>http://imageresizer.codeplex.com/releases/61778</id>
    <title type="text">3.0.0.0</title>
    <updated>2011-02-28T00:00:00Z</updated>
    <link href="http://imageresizer.codeplex.com/releases/view/61778" />
    <category term="Alpha" />
  </entry>
</feed>
{% endhighlight %}

The release date maps to the updated element, the version maps to the title, the download link is ...a link (duh), and
the release status maps to a category. Nice and simple, and 100% [Atom compliant][4].

Now the question was where should I host it? I briefly considered started a new Blogger blog (I saw another project
doing this), but it just seemed like overkill. Ultimately, I decided it would be best to host it on CodePlex, but how? A
hidden release? A wiki attachment? No, none of these felt right. I know, the source code repository! All the code files
are publicly accessible plus easy to maintain.

Luckily, both Subversion and Mercurial are HTTP-based and have ways of retrieving a raw file. Here are what the URLs
would look like:

| VCS        | URL                                                              |
| ---------- | ---------------------------------------------------------------- |
| Subversion | https://imageresizer.svn.codeplex.com/svn/releases.xml           |
| Mercurial  | https://hg01.codeplex.com/imageresizer/raw-file/tip/releases.xml |

Now it's time to write the code. First, I wrote some enums that would allow users to filter the kinds of updates they
wanted to be notified for.

{% highlight csharp %}
enum ReleaseStatus
{
    Stable = 1,
    Beta = 2,
    Alpha = 4
}

enum UpdateFilter
{
    Stable = ReleaseStatus.Stable,
    Beta = Stable | ReleaseStatus.Beta,
    Alpha = Beta | ReleaseStatus.Alpha
}
{% endhighlight %}

The status enum values are powers of two so that they can be combined to form the filters. The filters just add on top
of each other so that a filter of *Alpha* will update to a release status of *Stable*, *Beta*, or *Alpha*.

Finally, here is the component that allows an application to check for updates:

{% highlight csharp %}
class UpdateChecker
{
    public void CheckForUpdates(string feedUrl, UpdateFilter filter)
    {
        // NOTE: Requires a reference to System.ServiceModel.dll
        var formatter = new Atom10FeedFormatter();

        // Read the feed
        var reader = XmlReader.Create(feedUrl);
        formatter.ReadFrom(reader);

        // Get the version specified in AssemblyInfo.cs
        var currentVersion = Assembly.GetExecutingAssembly().GetName().Version;

        // Linq magic
        var update = (from i in formatter.Feed.Items
                      where new Version(i.Title.Text) > currentVersion &&
                          i.Categories.Any(c => CanUpgradeTo(c.Name, filter))
                      order by i.LastUpdatedTime descending).FirstOrDefault();

        if (update != null)
        {
            // TODO: Notify user of available download
            var downloadUrl = update.Links.Single().Uri.ToString();
        }
    }

    bool CanUpgradeTo(string status, UpdateFilter filter)
    {
        // Bitwise magic
        return (int)filter & (int)Enum.Parse(typeof(UpdateFilter), status, true) != 0;
    }
}
{% endhighlight %}

For my actual implementation, see [UpdaterService.cs][5].

So there you have it - a clean and maintainable way of implementing automatic updates on your CodePlex project. Please
feel free to contact me if you have any further questions.


  [1]: http://imageresizer.codeplex.com
  [2]: http://codeplex.codeplex.com/wikipage?title=CodePlexWebServices
  [3]: http://codeplex.codeplex.com/workitem/25424
  [4]: http://www.atomenabled.org/developers/syndication/atom-format-spec.php
  [5]: http://imageresizer.codeplex.com/SourceControl/changeset/view/63279#1142619
