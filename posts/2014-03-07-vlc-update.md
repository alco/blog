Let's talk about the user experience of VLC's update process on OS X.

This is how it currently works. I want to watch a video, so I locate the file via Finder and open it. VLC launches and update prompt appears in a dialog. If I click on the "Update" button, it'll start downloading the update. Then I have to click on the "Install and relaunch" button to upgrade to the newly downloaded version of the program.

All of the above is happening with the video already playing. But after the program is relaunched, it simply shows the blank VLC window where you have to open the video file again to finally be able to watch it.

## How it should have been done ##

Here's one way how the UX could be improved.

The program would still display the update dialog on launch. Afterwards, once the new binary had been downloaded, instead of the single "Install and relaunch" button it would show two buttons: "Install now" and "OK". Or as an alternative to the "OK" button VLC could display a notification saying that the update has been successfully downloaded.

The user is allowed to forget about the dialog altogether after they have clicked on the "Update" button. Next time they launch VLC, it'll be running the upgraded version. As simple as it seems, this is not how VLC works right now – you have to click the "Install" button after the update has been downloaded, otherwise you'll get the update prompt on the next launch.

And that's basically it. Those users who have already started watching a video don't care about the update at that point in time: the video is working, right? So shut up and let me watch it.

Those who need the update to make their video work will simply wait for it to download and click on the "Install now" button which would relaunch the program _and_ open the video again automatically (with no further input from the user that is).

---

Date: Fri Mar  7 00:28:27 EET 2014
