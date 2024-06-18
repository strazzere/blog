---
title: Vending.apk aka Market.apk
tags:
  - android
  - g1
  - market
  - market.apk
  - no dex?
  - vending.apk
url: vendingapk-aka-marketapk.html
id: 125
categories:
  - android
  - other
  - reverse engineering
date: 2008-12-03 21:58:32
---

Found this post I ment to put up yesterday, it was actually on an usb drive but I figured I'd throw it up here incase anyone else wanted the information... Enjoy!

Been playing around with the Vending.apk - which is present only on the G1 device currently, not on the emulator. If you've been following any of the threads over at xda-dev you might have seen one discussing how to move the market's cache to the sdcard. When I re-find this thread I will link it, but for now that's besides the point. I'm sure other people have been looking for a "market.apk" though after looking at this thread you'll realize that it is actually "Vending.apk". Though oddly enough, this .apk does not contain a dalvik-executable (.dex file), though it contains other files that you would normally find in .apk's.
The structure of the Vending.apk from my phone was the following;

```
+ Vending.apk
|
|–+ META-INF
| |
| |– CERT.RSA
| |– CERT.SF
| — MANIFEST.MF
|
|–+res
| |
| |–+ drawable
| | |
| | |– applications.png
| | |– border_youtube_thumbnail.9.png
| | |– border_youtube_thumbnail_pressed.9.png
| | |– border_youtube_thumbnail_selected.9.png
| | |– bottombar_portrait_565.png
| | |– btn_plus_default.png
| | |– checkout_button.png
| | |– dark_header.9.png
| | |– dark_header_dithered.xml
| | |– dots.png
| | |– dotted_line_480px.png
| | |– google_checkout_logo.png
| | |– ic_dialog_flag.png
| | |– ic_dialog_rate.png
| | |– ic_launcher_androidmarket.png
| | |– ic_maps_preferences.png
| | |– ic_menu_clear_rating.png
| | |– ic_menu_flag_content.png
| | |– ic_menu_home.png
| | |– ic_menu_info_details.png
| | |– ic_menu_my_downloads.png
| | |– ic_menu_search.png
| | |– ic_menu_security_setting.png
| | |– ic_menu_stop.png
| | |– ic_vm_application.png
| | |– ic_vm_games.png
| | |– ic_vm_my_downloads.png
| | |– ic_vm_search.png
| | |– ic_vm_thumbnail_big.png
| | |– ic_youtube_add_favorite.png
| | |– menu_favorites.png
| | |– menu_recents.png
| | |– ringtones.png
| | |– star_empty.png
| | |– star_full.png
| | |– star_half.png
| | |– stat_sys_install_complete.png
| | |– thumbnail_background.xml
| | |– vending.png
| | |– vending_small.png
| | |– wallpapers.png
| | |– youtube_carousel_big_default.9.png
| | |– youtube_carousel_big_pressed.9.png
| | |– youtube_carousel_big_selected.9.png
| | |– youtube_carousel_small_default.png
| | |– youtube_carousel_small_pressed.png
| | — youtube_carousel_small_selected.png
| |
| |–+ drawable-finger
| | |
| | |– ic_network_error.png
| | |– ic_tab_newest.xml
| | |– ic_tab_newest_selected.png
| | |– ic_tab_newest_unselected.png
| | |– ic_tab_popular.xml
| | |– ic_tab_popular_selected.png
| | — ic_tab_popular_unselected.png
| |
| |–+ layout
| | |
| | |– asset_browser_carousel.xml
| | |– asset_browser_list_item.xml
| | |– asset_category_browser.xml
| | |– asset_info.xml
| | |– asset_info_description.xml
| | |– asset_info_download_progress.xml
| | |– asset_info_downloads_ratings_header.xml
| | |– asset_info_my_review.xml
| | |– asset_info_section_header.xml
| | |– asset_info_simple_1_medium.xml
| | |– asset_info_simple_2_medium.xml
| | |– asset_list.xml
| | |– asset_list_footer.xml
| | |– asset_list_snippet.xml
| | |– asset_snippet.xml
| | |– asset_thumbnail.xml
| | |– buypage.xml
| | |– combined_results_footer.xml
| | |– combined_results_header.xml
| | |– combined_results_missing.xml
| | |– comment_list_item.xml
| | |– comments_activity.xml
| | |– confirm_permissions.xml
| | |– download_progress.xml
| | |– flag_content.xml
| | |– flag_content_message.xml
| | |– fullscreen_loading_indicator.xml
| | |– my_downloads_activity.xml
| | |– search_no_results.xml
| | |– set_rating.xml
| | |– tos.xml
| | |– uninstall_application.xml
| | — write_review.xml
| |
| |–+ menu
| | |
| | — base.xml
| |
| |–+ xml
| | |
| | |– searchable.xml
| |
|
|– AndroidManifest.xml
— resources.arsc
```

Looks like they have includes some interesting things, _buypage.xml_, _AndroidManifest.xml_ (this includes the intents and permissions the applications uses). I tried like many other people I'm sure, installing "Vending.apk" onto the emulator, but you will get a `Failure [-11]` when pushing it through adb.