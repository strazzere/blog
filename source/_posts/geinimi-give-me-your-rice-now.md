---
title: Geinimi - Give me your rice... NOW!
tags:
  - android
  - detection
  - geinimi
  - google
  - lookout
  - nmap
  - reverse engineering
  - teardown
  - trojan
  - virus
url: geinimi-give-me-your-rice-now.html
id: 386
categories:
  - android
date: 2011-01-07 16:57:17
---

So lately I've had the pleasure of reversing the latest Android threat, [Geinimi](http://blog.mylookout.com/2010/12/geinimi_trojan/), and boy - it sure is something else!

I don't want to go too much into depth for it here as it's already been torn apart and posted on my works blog. Our [latest release](http://blog.mylookout.com/2011/01/geinimi-trojan-technical-analysis/) has all the technical details outlined and a [full teardown in pdf form](http://blog.mylookout.com/_media/Geinimi_Trojan_Teardown.pdf).

Something I will post here is a nerfed script from Jaime Blasco, a newly found friend of mine who works for AlienVault. He posted a [nifty nmap script](http://www.alienvault.com/blog/jaime/2011/Jan/04) for detecting Geinimi infected devices. His original script can be found on his blog post, though I'm done some minor improvements to it;

```
description = [[
Detect if Geinimi trojan is present
]]

---
-- @output
-- PORT   STATE SERVICE
-- 5432/tcp open  postgresql
-- |_geinimi: Geinimi trojan present
-- 8791/tcp open
-- |_geinimi: Geinimi CMD plugin found and accepting commands

description = "Scan for Geinimi Trojan"

author = "(Original) - Jaime Blasco jaime.blasco@alienvault.com (Updates) - Tim Strazzere tim.strazzere@mylookout.com"

license = "Same as Nmap--See http://nmap.org/book/man-legal.html"

categories = {"discovery", "safe"}

require "comm"
require "shortport"
local stdnse = require "stdnse"

portrule = shortport.portnumber({5432, 4501, 6543, 8791}, {"tcp"})

action = function(host, port)
       local try = nmap.new_try()
       if (port.number == 8791) then
          local response = try(comm.exchange(host, port, "hi,xiaolu", {lines=100, proto=port.protocol, timeout=5000}))
          if (response:find "hi,liqian") then
             local response = try(comm.exchange(host, port, "CMD id", {lines=100, proto=port.protocol, timeout=5000}))
             if (response:find "command ok") then
                try(comm.exchange(host, port, "bye", {lines=100, proto=port.protocol, timeout=5000}))
                return "Geinimi CMD plugin found and accepting commands"
             else
                return "Geinimi CMD plugin found, but not accepting commands"
             end
          end
       else
          local response = try(comm.exchange(host, port, "hi,are you online?", {lines=100, proto=port.protocol, timeout=5000}))
          if (response:find "yes,I'm online!") then
             return "Geinimi trojan present"
          end
       end
end
```

Essentially I've added the three extra ports that the main Geinimi service will bind too. The main port is still 5432, though it can attempt to connect on the other two ports. The other thing I've added was a check to the presumed command plugin that a Geinimi device may also have, which is bound on port 8791.

Hopefully everyone enjoys the teardown and maybe this script would be useful for anyone who is interested in auditing their network of multiple Android devices.