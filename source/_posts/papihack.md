---
title: papihack?!
tags:
  - android
  - bytecode
  - games
  - high score
  - java
  - papicheat
  - papihack
  - papihack.java
  - papijump
  - papipwn
  - papiriver
  - reverse engineering
url: papihack.html
id: 167
categories:
  - android
date: 2009-01-20 17:21:14
---

![Oh no, papihack!](http://173.230.150.16/blog/wp-content/uploads/2009/01/papijump-04-200x300.png "13371337?! Oh no! papihack!")
So haven't been around for a while, due to the flu. So it gave me plenty of time to play PapiJump and PapiRiver. Though I kind of got interested in the method of high scores so I took a more indepth look at it. I'll posted all the code to emulator a papi* score submit. I took out the little tidbit that actually sends the request, so this just prints to the screen. I've also removed the "secret keys" which wouldn't be too hard to find if your _really_ wanted to use this.
```
/*
 * @file papihack.java
 *
 * @author Tim Strazzere
 *
 * @date Jan 20th, 2009 (finished same day)
 *
 * @desc:
 *    An attempt to spoof the high score function of papijump,
 *    and papiriver since they are just different secret keys
 *
 *    should produce a url similar to:
 *
 *    http://www.sunflat.net/android/cmd/postSc?gid=2001&v=1.0.1&lid=1&tid=4232145dfg8432145&dt=1232488839&sc=39927&ha=1007200355&tt=1&tn=T-Mobile+G1+Vi116143
 *    [broken down]
 *    http://www.sunflat.net/android/cmd/postSc?
 *    // game id
 *    gid=2000
 *    // game version
 *    &v=1.0.0
 *    // license?
 *    &lid=1
 *    // terminal id
 *    &tid=4232145dfg8432145
 *    // date / 1000
 *    &dt=1232171270
 *
 *    // score?
 *    &sc=844
 *    // CRC'ed hash
 *    &ha=3817043337
 *
 *    // eh? always 1
 *    &tt=1
 *    // terminal name
 *    &tn=T-Mobile+G1+Vi116143
 *
 */

import java.io.UnsupportedEncodingException;
import java.util.Date;
import java.util.zip.CRC32;
import java.util.zip.Checksum;

public class papihack {
	public static void main(String[] args) throws UnsupportedEncodingException{
		String comma = ",";
		//String appVersion = "1.0.0"; // version for papijump
		String appVersion = "1.0.1"; // version for papiriver
		//String secretKey = "6977616e74746f6368656174"; // secret key for papijump
		String secretKey = "69276d616368656174657221"; // secret key for papriver
		String terminalName = "T-Mobile G1 Vi116143"; // (Build.Model + " Vi" + Build.Version.Incremental) (needs to be websafe)
		String terminalID = "4232145dfg8432145"; // (Android_ID)
		//String gameID = "2000"; // papijump game id
		String gameID = "2001"; // papiriver game id
		String lid = "1";
		String score = "39927"; // score you want
		String postUrl = "http://www.sunflat.net/android/cmd/postSc?";

		Date date = new Date();
    	long dateLong = date.getTime()/1000;

    	StringBuilder hash = new StringBuilder();
    	hash.append(gameID + comma
    			+ appVersion + comma
    			+ lid + comma
    			+ terminalID + comma
    			+ dateLong + comma
    			+ secretKey + comma
    			+ score);
    	StringBuilder newHash = new StringBuilder(hash.toString().valueOf(hash));

    	newHash.append(",tt1");

    	Checksum checksum = new CRC32();
    	byte[] bytes = new byte[1024];
    	int len = newHash.toString().length();
    	Long value;

    	bytes = newHash.toString().getBytes("UTF-8");
    	checksum.update(bytes, 0, len);
    	value = checksum.getValue();

    	StringBuilder url = new StringBuilder();
    	url.append(postUrl);
    	url.append("gid=" + gameID);
    	url.append("v=" + appVersion);
    	url.append("lid=" + lid);
    	url.append("tid=" + terminalID);
    	url.append("dt=" + dateLong);
    	url.append("sc=" + score);
    	url.append("ha=" + value.toString());
    	url.append("tt=1"); // const ?
    	url.append("tn=" + terminalName);


    	System.out.println(url.toString());

	}
}
```
Ah, also - kudos to anyone who figures out what I replaced the keys with. It's sort of a little joke...

On a little after thought, maybe it should be called papipwn? Eh - oh well, either way it will be easily bannable/removable by the administrators. It's not hard as they are linked to your specific android id so I wouldn't recommend using this or if you do, going over board with it.