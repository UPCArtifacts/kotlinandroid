# An Empirical Study on Quality of Android Applications written in Kotlin language<a name="top"></a>
Here you can find the steps needed to replicate our study.

---

## Table of contents 

1. [Creation of a dataset of Kotlin applications](#datasetconstruction)
	* [F-Droid](#fdroid)
	* [AndroidTimeMachine](#androidtimemachine)
	* [AndroZoo](#androzoo)
2. [Filtering Kotlin applications](#filtering)
	* [Getting applications from 2017 and 2018](#recentapps)
	* [Applying our heuristics to find Kotlin applications](#hs)
3. [Code evolution of Android applications](#codeevolution)
4. [Analyzing the difference between Kotlin and Java applications in terms of presence of code smells](#codesmells)
5. [Detecting changes in the quality evolution trends after introducing Kotlin](#quality)

---

### 1. Creation of a dataset of Kotlin applications <a name="datasetconstruction"></a>


To study the using of Kotlin in the context of mobile development, we need first to select a dataset of mobile applications. In this work, we combined three well-known datasets, [F-droid](http://f-droid.org), [AndroZoo](http://androzoo.uni.lu) and [AndroidTimeMachine](https://androidtimemachine.github.io), to create our dataset.

#### F-Droid <a name="fdroid"></a>

**F-droid** is a directory of open-source Android applications. All apps listed in this directory are compiled from source and code repositories are publicly linked. In total, F-droid has **1509** applications on **06/04/2018**.

The table below shows some of these applications. To see the complete list click [here](docs/fdroid_all.md).

| App Name| Summary| Source-code Repo|
|---|---|---|
| 1. Aard| Offline dictionary| https://github.com/aarddict/android |
| 2. A2DP Volume| Manage Bluetooth functions| https://github.com/jroal/a2dpvolume |
| 3. A Time Tracker| Time Tracker| https://github.com/netmackan/ATimeTracker |
| 4. A Photo Manager| Manage local photos: Find/Copy/Edit-Exif and show in Gallery or Map.| https://github.com/k3b/androFotoFinder |
| 5. 34C3 Schedule| Schedule (aka FahrPlan) of the 34C3| https://github.com/EventFahrplan/EventFahrplan |
| 6. 33c3 Wifi Setup| Create secure Wifi connection entry for 33c3| https://github.com/eqvinox/wifisetup |
| 7. 33c3 SCR| Resolve Schedule conflicts| https://github.com/ligi/SCR |
| 8. 33C3 Schedule| Schedule (aka FahrPlan) of the 33C3| https://github.com/ligi/CampFahrplan |

F-droid provides on the main page of each application a link to download its last three versions and a link for another page with some technical information that contains all versions. 
**[Our crawler](https://github.com/brunomateus/f-droid-crawler)** is able to visit both pages for each application and retrieve the link to the apk for every version available. 


[back to the top](#top)

#### AndroidTimeMachine <a name="androidtimemachine"></a>

**AndroidTimeMachine** is a graph database of  Android apps which are both accessible on GitHub and Google.

In total, AndroidTimeMachine has **8,431** applications and it is based on publicly-available GitHub mirror available in [BigQuery](https://cloud.google.com/bigquery/public-data/github) dated on 18th October 2018.

The table below shows some of these applications. To see the complete visit the [official website](https://androidtimemachine.github.io).

| Source-code repo| Package name|
|---|---|
|1. https://github.com/learn-mobile-16/QuakeBuddy |com.gumgoose.app.quakebuddy |
|2. https://github.com/bigeyessolution/FestivalEdesio |com.bigeyessolution.FestivalEdesio |
|3. https://github.com/tarek360/PlayPauseDrawable |com.tarek360.playpausedrawable |
|4. https://github.com/gwhiteside/abstract-art |net.georgewhiteside.android.abstractart |
|5. https://github.com/shaun2029/ClockControl |uk.co.immutablefix.ClockControl |
|6. https://github.com/eziosoft/MultiWii_EZ_GUI |com.ezio.multiwii |
|7. https://github.com/jakemoritz/Tasking |me.jakemoritz.tasking_new | 
|8. https://github.com/meghalagrawal/NightSight |meghal.developer.nightsight.project |
|9. https://github.com/abbott221/WebsiteAndApp |com.MichaelAbbott.hexagonalgame |

This dataset is available in two different dockers: (i) first, containing the Neo4j database and (ii) second, containing a snapshot of all GitHub repositories in the dataset cloned to a local Gitlab.

[back to the top](#top)

#### AndroZoo <a name="androzoo"></a>

**AndroZoo** is dataset of millions of Android apps collected from various data sources using different web crawlers. The sources from which AndroZoo draws include major market places *Google Play, Anzhi, and AppChine*, as well as smaller directories mobile, *AnGeeks, Slideme, ProAndroid, HiApk*, and *F-Droid*.

In total, AndroZoo has **4390288** applications and **7795372** apks on **16/10/2018**.

The table below shows some of these applications. Each line corresponds to one apk file To see the complete list click [here](docs/latest.csv).

|Package name|Version code|Markets|Apk size in bytes|
|---|---|---|---|
|com.zte.bamachaye|121|anzhi|10386469|
|com.deperu.sitiosarequipa|10000|play.google.com|4300370|
|bmthx.god102409paperi|6|appchina|1044597|
|kr.ac.snjc.library|3|play.google.com|1375862|
|com.rbsoftware.pfm.personalfinancemanager|48|play.google.com|5345848|
|com.indoorway.android.fluctus|14|play.google.com|12798854|
|com.fearless.teengirl|3|play.google.com|4727630|
|com.appautomatic.ankulua.trial|39|play.google.com|16004883|
|com.kbf.app27730661|70101|play.google.com|1882706|

On the AndroZoo website the list of available APKs are regularly updated, along with metadata on each app as, the main package name, the size of the APK, the version, and the market where the app was downloaded from, etc

Furthermore, the AndroZoo's HTTP API allows to download full and unaltered APKs. 
[back to the top](#top)

For our experiment, we need the source-code and binary file (apk) from Android applications. F-droid provides both information, it contains only open-source apps and for each application it has a link to a public available source code repository (e.g., to GitHub) and the bytecode (apks) from different released versions and stores the previous versions of each app, which is necessary for studying the app evolution. However, F-Droid is small dataset if compared with AndroZoo. On the other hand, AndroZoo provides different released versions from each app, possibly from different stores, but it does not provide any information about applications' source-code. For that reason, we combined AndroZoo with AndroidTimeMachine. AndroidTimeMachine is considerable bigger than F-droid and it contains only open-source applications published on Google Play and hosted on GitHub, but it does not provide application's binary. Then, we use the application id(package name) to get the intersections between AndroZoo and AndroidTimeMachine, in this way we are able to get the source-code and apks from the set intersection. Therefore, our final dataset, as Figure below shows, corresponds to:  

**F-droid &cup; ( AndroZoo &cap; AndroidTimeMachine)**

![Final dataset](docs/final_dataset.png)

[back to the top](#top)

---

### 2. Filtering Kotlin applications <a name="filtering"></a>

In this work, we are interested in a subset of mobile applications from F-droid, AndroidTimeMachine and AndroZoo: those that contain Kotlin code.

As Kotlin was announced as an official language for Android development in 2017, before that date, Android developers did not have support from Google for developing Android apps using Kotlin language.

Therefore, we consider that apps which the last versions date from 2016 or earlier could not give us much information about the use of Kotlin language in the Android domain.
Consequently, the criterion we used for building our dataset of mobile apps is to select every application whose last version was released in 2017 or later.

#### Getting applications from 2017 and 2018 <a name="recentapps"></a>


##### From F-Droid

Using the upload version provided by F-Droid, total number of applications retrieved from F-Droid by the date of June 4th, 2018 that fulfill our selection criterion is 928. 

The table below shows some of these applications. To see the complete list click [here](docs/fdroid_2017_2018.md).

|App name| Pakcage Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|1.A Photo Manager|de.k3b.android.androFotoFinder|Manage local photos: Find/Copy/Edit-Exif and show in Gallery or Map.|2018-03-23|https://github.com/k3b/androFotoFinder|
|2.34C3 Schedule|info.metadude.android.congress.schedule|Schedule (aka FahrPlan) of the 34C3|2018-01-03|https://github.com/EventFahrplan/EventFahrplan|
|3.1010! Klooni|io.github.lonamiwebs.klooni|libGDX game based on 1010|2018-02-06|https://github.com/LonamiWebs/Klooni1010|
|4.Additional Security Settings|com.davidshewitt.admincontrol|Additional security settings|2018-03-23|https://github.com/linux-colonel/AdminControl|
|5.AnySoftKeyboard - French Language Pack|com.anysoftkeyboard.languagepack.french|AnySoftKeyboard French Language pack|2018-03-23|https://github.com/AnySoftKeyboard/LanguagePack/tree/French|
|6.AnySoftKeyboard|com.menny.android.anysoftkeyboard|Alternative keyboard|2018-03-05|https://github.com/AnySoftKeyboard/AnySoftKeyboard|
|7.AntennaPod|de.danoeh.antennapod|Advanced podcast manager and player|2018-04-16|https://github.com/antennapod/AntennaPod|
|8.Another RSS|de.digisocken.anotherrss|reader for multiple RSS feeds in a single, searchable list|2018-05-23|https://github.com/no-go/AnotherRSS|

##### From AndroidTimeMachine and AndroZoo

Originally, AndroidTimeMachine contains **8431** and the most recent apps contained is from October 2017, since the first step of building this dataset, i.e, query  the publicly-available GitHub mirror available in BigQuery, was executed considering a mirror from that date. In order to get more recent apps and potentially more Kotlin apps, we re-executed the steps needed to create [AndroidTimeMachine](https://androidtimemachine.github.io/dataset/), using the [tool] (https://github.com/AndroidTimeMachine/open_source_android_apps) provided by the authors. 

**Step executed:**

1. Query GitHub data on BigQuery on 18/10/2018 to Find Android Manifest Files 
 - **358,518**  found
 - Includes many build artifacts, included libraries,
2. Extracting Packages names from each Manifest
 - **114,152** unique packages
3. Extracting Package names from repositories with only one manifest file (Avoid false/positive)
 - **49,570** unique packages
4. Filtering for Apps on Google Play
 - **3664** found
5. Matching of Google Play pages to GitHub repositories
 - **3664** found

The table below shows the false/positive avoided by our incluision of a new step, step 3.

|Repo URL|Package|App on Google Play| 
|---|---|---|
[FinalHerramientas-2014-2](https://github.com/afmurillo/FinalHerramientas-2014-2)| com.google.android.apps.googlevoice|[Google Voice](https://play.google.com/store/apps/details?id=com.google.android.apps.googlevoice)|
[FinalHerramientas-2014-2](https://github.com/afmurillo/FinalHerramientas-2014-2)| com.google.android.gm|[Gmail](https://play.google.com/store/apps/details?id=com.google.android.gm)|
[MIUI-XML-6.0-Indonesian](https://github.com/bamz3r/MIUI-XML-6.0-Indonesian)|com.google.android.music|[Google Music](https://play.google.com/store/apps/details?id=com.google.android.music)|
[react-native-navigation](https://github.com/travelbird/react-native-navigation)|com.airbnb.android|[Airbnb](https://play.google.com/store/apps/details?id=com.airbnb.android)|
|[foursquared.eclair](https://github.com/sbolisetty/foursquared.eclair)|com.joelapenna.foursquared|[Foursquare](https://play.google.com/store/apps/details?id=com.joelapenna.foursquared)|

Using the metadata downloaded from the app store on step 4, more specifically, the field **'updateDate'** that contains the date when the applications was last update, we filtered applications whose were update in 2017 and later. We found **2156** applications that fulfill our selection criterion.

The table below shows some of these applications. To see the complete list click [here](docs/androidtimemachine_2017_2018.md).

App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|1. 10,000 sentences|info.puzz.a10000sentences|10,000 sentences will help you learn new words in languages you are learning.|Jan 27, 2018|https://github.com/tkrajina/10000sentences|
|2. 10-bit Clock Widget|com.github.ashutoshgngwr.tenbitclockwidget|View your home screen clock in bits.|Oct 22, 2017|https://github.com/ashutoshgngwr/10-bitClockWidget|
|3. 104 Birds Quiz|net.project104.civyshkbirds|Game to identify birds from their picture or their singings|Jun 3, 2017|https://github.com/civyshk/104birds|
|4. 104 RPN Calc|net.project104.civyshkrpncalc|Calc that successfully merges the Reverse Polish Notation and the Infix Notation|Jun 19, 2018|https://github.com/CIVYSH/rpncalc104k|
|5. 106/109ハードウェアキーボード配列変更(+親指Ctrl)|jp.kcm.thumbctrl|日本語ハードウェアキーボード用に日本語配列を追加. ([半/全]と[ESC]入替、[変換]/[無変換]→[Ctrl]、[Caps]→[半/全]も追加されます)|Jun 13, 2018|https://github.com/shiftrot/ThumbCtrl|
|6. 10Sel|com.app.android.tensel|peer to peer mobile commerce. Sell and request stuff with peers around you|Sep 7, 2017|https://github.com/Nguedia-Adele/J_Trok|
|7. 15 Puzzle Game - Photo Edition|de.fintasys.the_15_puzzle_game|Use your photos from Instagram or from your device to play the 15-Puzzle-Game!|Mar 30, 2017|https://github.com/Fintasys/15-Puzzle-Game|
|8. 2048 (Ads Free)|com.tpcstld.twozerogame|Port of Gabriele Cirulli's 2048 to Android.\n\n+No Ads!\n+No permissions!|Jun 10, 2018|https://github.com/BuddyBuild/2048-Android|
|9. 5 Calls|org.a5calls.android.a5calls|Make your voice heard: Spend 5 minutes, make 5 calls.|Jul 18, 2018|https://github.com/bryansills/5calls-android|
|10. 5 Pin Bowling Companion|ca.josephroque.bowlingcompanion|Track 5 pin bowling scores frame-by-frame, along with a medley of statistics!|Sep 29, 2018|https://github.com/sunndog/bowling-companion|

Using the packages names of each app, we queried the csv file provided by AndroZoo to identify which applications we could download the apks. Then, we found **236** applications present on F-Droid as well, remaining  **1295**  applications  to  download. Then, we  checked  manually  for  each  of  1295  applications, its pages on GooglePlay and its source-code link, to avoid applications liked with wrong repositories, including applications not open-source, because this would result on downloading wrong apks. We found and removed 54 applications, including some not open-source apps, as Facebook Mensseger  and Twitter.  The  complete  list  of  apps  removed  is  available [here](docs/wrong_match.md).


The table below shows some of these applications. To see the complete list click [here](docs/androidtimemachine_androzoo.md) and to see the intersection between the datasets click [here](docs/intersection.md).

|App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|1. 10,000 sentences|info.puzz.a10000sentences|10,000 sentences will help you learn new words in languages you are learning.|Jan 27, 2018|https://github.com/tkrajina/10000sentences|
|2. 104 RPN Calc|net.project104.civyshkrpncalc|Calc that successfully merges the Reverse Polish Notation and the Infix Notation|Jun 19, 2018|https://github.com/CIVYSH/rpncalc104k|
|3. 106/109ハードウェアキーボード配列変更(+親指Ctrl)|jp.kcm.thumbctrl|日本語ハードウェアキーボード用に日本語配列を追加. ([半/全]と[ESC]入替、[変換]/[無変換]→[Ctrl]、[Caps]→[半/全]も追加されます)|Jun 13, 2018|https://github.com/shiftrot/ThumbCtrl|
|4. 2048 (Ads Free)|com.tpcstld.twozerogame|Port of Gabriele Cirulli's 2048 to Android.\n\n+No Ads!\n+No permissions!|Jun 10, 2018|https://github.com/BuddyBuild/2048-Android|
|5. 5 Calls|org.a5calls.android.a5calls|Make your voice heard: Spend 5 minutes, make 5 calls.|Jul 18, 2018|https://github.com/bryansills/5calls-android|
|6. 5 Pin Bowling Companion|ca.josephroque.bowlingcompanion|Track 5 pin bowling scores frame-by-frame, along with a medley of statistics!|Sep 29, 2018|https://github.com/sunndog/bowling-companion|
|7. A Mohammed Khaleel’s Invitation|com.slbdev.demo|A Mohammed Khaleel’s Invitation|Aug 9, 2018|https://github.com/alexrainman/CarouselView|
|8. A.scribe|com.laaidback.ascribe|open-source, minimalist note-taking app|Aug 3, 2017|https://github.com/Laaidback/A.scribe|

In total, our dataset should have **2169** (928 + 1241) applications, but we could not download any apks from two applications, then our final dataset has **2167** apps and **19838** apks.

The table below shows some of these applications. To see the complete list click [here](docs/final_dataset.md).

|App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|1. 10,000 sentences|info.puzz.a10000sentences|10,000 sentences will help you learn new words in languages you are learning.|Jan 27, 2018|https://github.com/tkrajina/10000sentences|
|2. 104 RPN Calc|net.project104.civyshkrpncalc|Calc that successfully merges the Reverse Polish Notation and the Infix Notation|Jun 19, 2018|https://github.com/CIVYSH/rpncalc104k|
|3. 106/109ハードウェアキーボード配列変更(+親指Ctrl)|jp.kcm.thumbctrl|日本語ハードウェアキーボード用に日本語配列を追加. ([半/全]と[ESC]入替、[変換]/[無変換]→[Ctrl]、[Caps]→[半/全]も追加されます)|Jun 13, 2018|https://github.com/shiftrot/ThumbCtrl|
|4. 2048 (Ads Free)|com.tpcstld.twozerogame|Port of Gabriele Cirulli's 2048 to Android.\n\n+No Ads!\n+No permissions!|Jun 10, 2018|https://github.com/BuddyBuild/2048-Android|
|5. 5 Calls|org.a5calls.android.a5calls|Make your voice heard: Spend 5 minutes, make 5 calls.|Jul 18, 2018|https://github.com/bryansills/5calls-android|
|6. 5 Pin Bowling Companion|ca.josephroque.bowlingcompanion|Track 5 pin bowling scores frame-by-frame, along with a medley of statistics!|Sep 29, 2018|https://github.com/sunndog/bowling-companion|
|7. A.scribe|com.laaidback.ascribe|open-source, minimalist note-taking app|Aug 3, 2017|https://github.com/Laaidback/A.scribe|
|8. AES Crypto|com.evgenii.aescrypto|Encrypt your message with one tap.|Sep 21, 2018|https://github.com/evgenyneu/aes-crypto-android|
|9. APC UPS Countdown|net.formula97.android.apcupscountdown|APC by Schneider ElectricのUPSで表示される時間情報の計算を補助する|Aug 10, 2018|https://github.com/f97one/ApcUpsCountdown|
|10. APK Downloader|ca.barraco.carlo.apkdownloader|Save apps installed on your device to an APK file in your Download folder.|Jun 28, 2018|https://github.com/cbarox/APK-Downloader|
[back to the top](#top)

#### Applying our heuristics to find Kotlin applications <a name="hs"></a>

We build a process to classify both applications and apks retrieved in two categories of applications: (1) written (partially or totally) with Kotlin, and (2) written with Java (not include any line of Kotlin code).

The Figure below shows the process for classifying applications between Kotlin and Java based, which has 3 main steps.

![Pipeline to classify android applications](docs/heuristics.png)

We  first apply a heuristics *H_apk*, Figure (a), that consists of looking for a folder called **kotlin** inside the apk file. To  automatize this task,  we use an Android developer tool provided by the Android SDK, named *apkanalyzer*. Using this heuristic, we first classify each version (apk) of an application. Then, if at least one apk of an application is classified as Kotlin, the heuristic  classifies  the application as 'Kotlin'. 
Otherwise, it classifies as 'Java'.
The *H_apk*  provides a cheap and fast approach to get an initial guess about the presence of Kotlin code.

At the same time, we apply our second heuristic *H_gh*, Figure(b), which relies on the GitHub API:  for each application hosted on GitHub, we query the GitHub API to retrieve the amount of code (expressed in bytes) from the most recent version (i.e., the HEAD) grouped by the programming language. We classify the app as `Kotlin' if Kotlin language is present in the response of the API. 

Finally, once we retrieve a set of candidate Kotlin applications using *H_apk* and *H_gh*, i.e, ***H\_apk &cup; H\_gh***, we apply the heuristic *H_sc*, Figure(c), to assert the presence of Kotlin code in, at least, one commit.  *H_sc* inspects every commit of the source code repository of an application.
An application is classified as Kotlin if the heuristic fin>s, at least, one commit that introduces Kotlin code. 
To carry out this task, the heuristic used the tool [CLOC](http://cloc.sourceforge.net/) which returns a list with the languages used in an application and the amount of code regarding no-blank lines. 

Our final dataset corresponds to ***(H\_apk &cup; H\_gh) &cap; H\_sc***.


##### Results

The Table below summarizes the classification of applications  done using the methodology  presented before.

|Information|Total|Kotlin|Java|
|---|---|---|---|
|Unique apps|2167|244|1923|
|Versions|19838|1590|18248|

Our dataset has **2167** applications: 
**244 (11.25%)** of them contain, at least, one version (apk) that includes Kotlin code. The rest of the applications, **1923 (88.75%)** do not have any version that contain Kotlin code. 
The Figure below shows these percentages.  Considering the number of versions (apk), we found 
**1590** apks **(8.0%)** with Kotlin code and **18248 (92,0%)** without Kotlin code. 

**Unique apps**
![Unique apps](docs/unique_apps.png)

**All versions**
![Unique apps](docs/apks.png)

Over a total of 2221 Kotlin applications:

  *  *H_aph* classified 265 app as 'Kotlin', i.e., those with at least one 'Kotlin' apk. The table below shows some of these applications. To see the complete list click [here](docs/hapk.md).
 
|App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|6. 5 Pin Bowling Companion|ca.josephroque.bowlingcompanion|Track 5 pin bowling scores frame-by-frame, along with a medley of statistics!|Sep 29, 2018|https://github.com/sunndog/bowling-companion|
|9. APC UPS Countdown|net.formula97.android.apcupscountdown|APC by Schneider ElectricのUPSで表示される時間情報の計算を補助する|Aug 10, 2018|https://github.com/f97one/ApcUpsCountdown|
|14. Accounting Fantozzi|com.eusecom.samfantozzi|Android mobil accounting Fantozzi complete accounting for smartphone and tablet.|Jun 21, 2018|https://github.com/eurosecom/samfantozzi|
|18. Adblock Fast|com.rocketshipapps.adblockfast|Adblock Fast is a free, open-source ad blocker for Samsung Internet 4.0 and up!|May 17, 2017|https://github.com/rocketshipapps/adblockfast|
|45. Ad-Free|ch.abertschi.adfree|AdBlocker for Spotify|2018-02-16|https://github.com/abertschi/ad-free|
|59. Always On AMOLED - BETA|com.tomer.alwayson|Get an always on display on your device!|May 20, 2018|https://github.com/rosenpin/AlwaysOnDisplayAmoled|
|79. Android Playground|com.esafirm.androidplayground|Android Playground|Jan 3, 2018|https://github.com/esafirm/android-playground|
|85. AnkiEditor for AnkiDroid|com.jkcarino.ankieditor|An advanced note editor plug-in for AnkiDroid|Sep 29, 2018|https://github.com/jkennethcarino/AnkiEditor|

 * *H_gh* classified 234 app as`Kotlin'. Furthermore, 41 out of 234 apps were not classified by *H_apk* as 'Kotlin'. However, we found [two repositories linked with two apps each](docs/sharing_repo.md). We manually checked these repositories and as consequence we removed two apps. For the apps linked with the [repo1](docs/sharing_repo.md#repo1), we realized that the authors used different packages names in two different stores, F-Droid and GooglePlay, for the same app. For the apps linked with [repo2](docs/sharing_repo.md#repo2), we found that one of them is a plugin for the main app and this plugin does not have Kotlin code. 

The table below shows some of these applications. To see the complete list click [here](docs/hgh.md).



|App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|
|28. 34C3 Schedule|info.metadude.android.congress.schedule|Schedule (aka FahrPlan) of the 34C3|2018-01-03|https://github.com/EventFahrplan/EventFahrplan|
|45. Ad-Free|ch.abertschi.adfree|AdBlocker for Spotify|2018-02-16|https://github.com/abertschi/ad-free|
|69. And Bible|net.bible.android.activity|Offline Bible reader|2017-04-13|https://github.com/mjdenham/and-bible|
|79. Android Playground|com.esafirm.androidplayground|Android Playground|Jan 3, 2018|https://github.com/esafirm/android-playground|
|85. AnkiEditor for AnkiDroid|com.jkcarino.ankieditor|An advanced note editor plug-in for AnkiDroid|Sep 29, 2018|https://github.com/jkennethcarino/AnkiEditor|
|119. Apk Analyzer|sk.styk.martin.apkanalyzer|Take a look under the hood of your applications!|Oct 16, 2018|https://github.com/MartinStyk/AndroidApkAnalyzer|
|125. Apolline|science.apolline|Application for air pollution monitoring|May 27, 2018|https://github.com/Apolline-Lille/apolline-android|
|127. App Launcher|com.simplemobiletools.applauncher|A simple holder for your favourite app launchers.|2018-05-23|https://github.com/SimpleMobileTools/Simple-App-Launcher|

* We tried to apply *H_sc* on 304 repositories (*H_apk &cup; H_gh*) but 7 repositories were not available. Then we analyzed 297 repositories and we found 244 repositories that contains Kotlin code.
To see the complete list click [here](docs/final_kotlin_repos.md). It is important to mentioned that two repositories appear twice.

|App name| Package Name| Description | Upload Date | Source-code repo|
|---|---|---|---|---|	
|6. 5 Pin Bowling Companion|ca.josephroque.bowlingcompanion|Track 5 pin bowling scores frame-by-frame, along with a medley of statistics!|Sep 29, 2018|https://github.com/sunndog/bowling-companion|
|9. APC UPS Countdown|net.formula97.android.apcupscountdown|APC by Schneider ElectricのUPSで表示される時間情報の計算を補助する|Aug 10, 2018|https://github.com/f97one/ApcUpsCountdown|
|14. Accounting Fantozzi|com.eusecom.samfantozzi|Android mobil accounting Fantozzi complete accounting for smartphone and tablet.|Jun 21, 2018|https://github.com/eurosecom/samfantozzi|
|18. Adblock Fast|com.rocketshipapps.adblockfast|Adblock Fast is a free, open-source ad blocker for Samsung Internet 4.0 and up!|May 17, 2017|https://github.com/rocketshipapps/adblockfast|
|28. 34C3 Schedule|info.metadude.android.congress.schedule|Schedule (aka FahrPlan) of the 34C3|2018-01-03|https://github.com/EventFahrplan/EventFahrplan|
|45. Ad-Free|ch.abertschi.adfree|AdBlocker for Spotify|2018-02-16|https://github.com/abertschi/ad-free|
|59. Always On AMOLED - BETA|com.tomer.alwayson|Get an always on display on your device!|May 20, 2018|https://github.com/rosenpin/AlwaysOnDisplayAmoled|

### 3. Code evolution of Android applications <a name="codeevolution"></a>

We classified each Kotlin application according to the code evolution trends presented on the following table:

|Source-code evolution|
|---|
| ET 1. Kotlin is the initial language and the amount of Kotlin grows|
| ET 2. Kotlin code replaces all Java code|
| ET 3. Kotlin code replaces some Java then  Java continues growing|
| ET 4. Kotlin increase together with Java|
| ET 5. Kotlin grows and Java decreases (but never is zero)|
| ET 6. Kotlin grows and Java decreases until the Java code is 0|
| ET 7. Kotlin grows and Java remains constant|
| ET 8. Kotlin is constant and Java changes|
| ET 9. Kotlin and Java remain constant|
| ET 10. Kotlin introduced but lately disappears|
| ET 11. Java replaces Kotlin code|
| ET 12. Other|


The following table shows the results.

|Source-code evolution| # Apps| % |
|---|---|---|
| ET 1. Kotlin is the initial language and the amount of Kotlin grows|19|7.8|
| ET 2. Kotlin code replaces all Java code|15|6.1|
| ET 3. Kotlin code replaces some Java then Java continues growing|4|1.6|
| ET 4. Kotlin increase together with Java|8|3.3|
| ET 5. Kotlin grows and Java decreases (but never is zero)|52|21.3|
| ET 6. Kotlin grows and Java decreases until the Java code is 0|48|19.7|
| ET 7. Kotlin grows and Java remains constant|41|16.8|
| ET 8. Kotlin is constant and Java changes|43|17.6|
| ET 9. Kotlin and Java remain constant|7|2.9|
| ET 10. Kotlin introduced but lately disappears|3|1.2|
| ET 11. Java replaces Kotlin code|2|0.8|
| ET 12. Other|2|0.8|
| Total of applications| 244| 100%|

The following figures display, for each code evolution trend, the code evolution of one particular application as example.

![ET 1: Kotlin is the initial language.](docs/evolution/et1/lines_Mygod-Vpnhotspot.png) ET 1: Kotlin is the initial language.

![ET 2: Kotlin code replaces all Java code.](docs/evolution/et2/lines_Simplemobiletools-Simple-Flashlight.png) ET 2: Kotlin code replaces all Java code.

![ET 3:  Kotlin code replaces some Java then Java continues growing.](docs/evolution/et3/lines_Maxr1998-Home-Assistant-Android.png)
ET 3:  Kotlin code replaces some Java then Java continues growing.

![ET 4: Kotlin increase together with Java.](docs/evolution/et4/lines_Streetwalrus-Android_Usb_Msd.png)
ET 4: Kotlin increase together with Java.

![ET 5: Kotlin grows and Java decreases.](docs/evolution/et5/lines_Thaapasa-Jalkametri-Android.png)
ET 5: Kotlin grows and Java decreases. Example 1.

![ET 5: Kotlin grows and Java decreases.](docs/evolution/et5/lines_Caarmen-Poet-Assistant.png)
ET 5: Kotlin grows and Java decreases. Example 2.

![ET 6: Kotlin grows and Java decrease until the Java code is 0.](docs/evolution/et6/lines_Simplemobiletools-Simple-Calendar.png)
ET 6: Kotlin grows and Java decrease until the Java code is 0.

![ET 7: Kotlin grows and Java remains constant.](docs/evolution/et7/lines_Apiote-Bimba.png)
ET 7: Kotlin grows and Java remains constant.

![ET 8: Kotlin is constant and Java grows.](docs/evolution/et8/lines_Nextcloud-Talk-Android.png)
ET 8: Kotlin is constant and Java grows.

![ET 9: Kotlin and Java remain constant.](docs/evolution/et9/lines_Wykcode-Wyk-Android.png)
ET 9: Kotlin and Java remain constant.

![ET 10: Kotlin introduced but lately disappears.](docs/evolution/et10/lines_Helloworld1-Freeotpplus.png)
ET 10: Kotlin introduced but lately disappears.

![ET 11:Java replaces Kotlin code.](docs/evolution/et11/lines_Garmax1-Material-Flashlight.png)
ET 11:Java replaces Kotlin code.

![ET 12: Other.](docs/evolution/et12/lines_Rosenpin-Quickdraweverywhere.png)
ET 12: Other.


To see all graphs click [here](docs/evolution/README.md)

### 4. Analyzing the difference between Kotlin and Java applications in terms of presence of code smells <a name="codesmells"></a>

<style type="text/css">
.tg .tg-baqh{text-align:center;vertical-align:top}
.tg .tg-0lax{text-align:left;vertical-align:top}
</style>

<table class="tg">
  <tr>
    <th class="tg-baqh" rowspan="4">Lang</th>
    <th class="tg-baqh" colspan="10">% Affected applications by smells</th>
  </tr>
  <tr>
    <td class="tg-baqh" colspan="4">Object-oriented smells</td>
    <td class="tg-baqh" colspan="6">Android Smells</td>
  </tr>
  <tr>
    <td class="tg-baqh">Long Method</td>
    <td class="tg-baqh">Complex Class</td>
    <td class="tg-baqh">Blob class</td>
    <td class="tg-baqh">Swiss Army Knife</td>
    <td class="tg-baqh">No Low Memory Resolver</td>
    <td class="tg-baqh">UI Overdraw</td>
    <td class="tg-baqh">Heavy Broadcast Receiver</td>
    <td class="tg-baqh">Heavy Service Start</td>
    <td class="tg-baqh">Heavy Async Task</td>
    <td class="tg-baqh">Init OnDraw</td>
  </tr>
  <tr>
    <td class="tg-baqh">LM</td>
    <td class="tg-baqh">CC</td>
    <td class="tg-baqh">BLOB</td>
    <td class="tg-baqh">SAK</td>
    <td class="tg-baqh">NLMR</td>
    <td class="tg-baqh">UIO</td>
    <td class="tg-baqh">HBR</td>
    <td class="tg-baqh">HSS</td>
    <td class="tg-baqh">HAS</td>
    <td class="tg-baqh">IOD</td>
  </tr>
  <tr>
    <td class="tg-0lax">Pure Kotlin</td>
    <td class="tg-baqh">99.80</td>
    <td class="tg-baqh">98.50</td>
    <td class="tg-baqh">95.12</td>
    <td class="tg-baqh">65.67</td>
    <td class="tg-baqh">99.30</td>
    <td class="tg-baqh">54.02</td>
    <td class="tg-baqh">35.72</td>
    <td class="tg-baqh">17.31</td>
    <td class="tg-baqh">09.55</td>
    <td class="tg-baqh">09.55</td>
  </tr>
  <tr>
    <td class="tg-0lax">Java</td>
    <td class="tg-baqh">99.62</td>
    <td class="tg-baqh">96.89</td>
    <td class="tg-baqh">93.53</td>
    <td class="tg-baqh">66.51</td>
    <td class="tg-baqh">98.84</td>
    <td class="tg-baqh">45.33</td>
    <td class="tg-baqh">39.98</td>
    <td class="tg-baqh">19.92</td>
    <td class="tg-baqh">22.61</td>
    <td class="tg-baqh">06.50</td>
  </tr>
  <tr>
    <td class="tg-0lax">K - J</td>
    <td class="tg-baqh">0.18</td>
    <td class="tg-baqh">1.61</td>
    <td class="tg-baqh">1.59</td>
    <td class="tg-baqh">-0.84</td>
    <td class="tg-baqh">0.46</td>
    <td class="tg-baqh">8.69</td>
    <td class="tg-baqh">-4.26</td>
    <td class="tg-baqh">-2.61</td>
    <td class="tg-baqh">-13.06</td>
    <td class="tg-baqh">3.05</td>
  </tr>
</table>

<table class="tg">
  <tr>
    <th class="tg-xldj" rowspan="2">Smell</th>
    <th class="tg-xldj" rowspan="2">Lang</th>
    <th class="tg-xldj">Median</th>
    <th class="tg-xldj" rowspan="2">Cliff's &delta;</th>
    <th class="tg-xldj">Significance of</th>
  </tr>
  <tr>
    <td class="tg-0pky">Ratio</td>
    <td class="tg-0pky">difference</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">LM</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0563</td>
    <td class="tg-0pky" rowspan="2">-0.3873</td>
    <td class="tg-0pky" rowspan="2">Medium</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0736</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">CC</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0593</td>
    <td class="tg-0pky" rowspan="2">-0.3168</td>
    <td class="tg-0pky" rowspan="2">Small</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0781</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">BLOB</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0163</td>
    <td class="tg-0pky" rowspan="2">-0.4338</td>
    <td class="tg-0pky" rowspan="2">Medium</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0278</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">SAK</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0008</td>
    <td class="tg-0pky" rowspan="2">-0.2433</td>
    <td class="tg-0pky" rowspan="2">Small</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0040</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">NLMR</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.3750</td>
    <td class="tg-0pky" rowspan="2">-0.2915</td>
    <td class="tg-0pky" rowspan="2">Small</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">1.0000</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">UIO</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0769</td>
    <td class="tg-0pky" rowspan="2">0.1156</td>
    <td class="tg-0pky" rowspan="2">Insignificant</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0000</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">HBR</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0000</td>
    <td class="tg-0pky" rowspan="2">-0.0699</td>
    <td class="tg-0pky" rowspan="2">Insignificant</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0000</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">HSS</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0000</td>
    <td class="tg-0pky" rowspan="2">-0.0240</td>
    <td class="tg-0pky" rowspan="2">Insignificant</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0000</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">HAS</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0000</td>
    <td class="tg-0pky" rowspan="2">-0.1306</td>
    <td class="tg-0pky" rowspan="2">Insignificant</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0000</td>
  </tr>
  <tr>
    <td class="tg-0pky" rowspan="2">IOD</td>
    <td class="tg-0pky">Kotlin</td>
    <td class="tg-0pky">0.0000</td>
    <td class="tg-0pky" rowspan="2">0.0341</td>
    <td class="tg-0pky" rowspan="2">Insignificant</td>
  </tr>
  <tr>
    <td class="tg-0pky">Java</td>
    <td class="tg-0pky">0.0000</td>
  </tr>
</table>

### 5. Detecting changes in the quality evolution trends after introducing Kotlin<a name="quality"></a>

<table class="tg">
  <tr>
    <td class="tg-uys7" rowspan="2">Code Smell</td>
    <td class="tg-uys7" colspan="2"># Apps Kotlin Improves Quality Score</td>
    <td class="tg-quj4" rowspan="2">Positive Change on Evolution Trend</td>
  </tr>
  <tr>
    <td class="tg-dvpl">First Kotlin</td>
    <td class="tg-c3ow">Last Kotlin</td>
  </tr>
  <tr>
    <td class="tg-c3ow">LM</td>
    <td class="tg-dvpl">28(50%)</td>
    <td class="tg-dvpl">28(50%)</td>
    <td class="tg-dvpl">10/28(35.71%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">CC</td>
    <td class="tg-dvpl">35(62.5%)</td>
    <td class="tg-dvpl">36(64.29%)</td>
    <td class="tg-dvpl">13/36(36.11%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">BLOB</td>
    <td class="tg-dvpl">43(76.79%)</td>
    <td class="tg-dvpl">42(75%)</td>
    <td class="tg-dvpl">18/42(42.86%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">SAK</td>
    <td class="tg-dvpl">44(78.57%)</td>
    <td class="tg-dvpl">45(80.36%)</td>
    <td class="tg-dvpl">17/45(37.78%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">HBR</td>
    <td class="tg-dvpl">45(80.36%)</td>
    <td class="tg-dvpl">40(71.43%)</td>
    <td class="tg-dvpl">7/40(17.5%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">HAS</td>
    <td class="tg-dvpl">37(66.07%)</td>
    <td class="tg-dvpl">30(53.57%)</td>
    <td class="tg-dvpl">2/30(6.67%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">HSS</td>
    <td class="tg-dvpl">45(80.36%)</td>
    <td class="tg-dvpl">42(75%)</td>
    <td class="tg-dvpl">6/42(14.29%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">IOD</td>
    <td class="tg-dvpl">36(64.29%)</td>
    <td class="tg-dvpl">36(64.29%)</td>
    <td class="tg-dvpl">5/36(13.89%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">NLMR</td>
    <td class="tg-dvpl">33(58.93%)</td>
    <td class="tg-dvpl">31(55.36%)</td>
    <td class="tg-dvpl">6/31(19.35%)</td>
  </tr>
  <tr>
    <td class="tg-c3ow">UIO</td>
    <td class="tg-dvpl">36(64.29%)</td>
    <td class="tg-dvpl">36(64.29%)</td>
    <td class="tg-dvpl">8/36(22.22%)</td>
  </tr>
</table>
