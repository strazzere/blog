---
title: '"VENDING_REQUIRE_SIM_FOR_PURCHASE"... Interesting...'
tags:
  - android
  - g1
  - google
  - market
  - market place
  - Vending
  - VENDING_REQUIRE_SIM_FOR_PURCHASE
  - VENDING_SYNC_FREQUENCY_MS
  - VENDING_TOS_VERSION
  - vending.apk
url: vending-require-sim-for-purchase-interesting.html
id: 214
categories:
  - android
date: 2009-03-08 21:47:08
---

So I was skimming through some code looking for some specifics when I ran across the following stuff in Settings;
```
        /**
         * Frequency in milliseconds at which we should sync the locally installed Vending Machine
         * content with the server.
         */
        public static final String VENDING_SYNC_FREQUENCY_MS = "vending_sync_frequency_ms";

        /**
         * Support URL that is opened in a browser when user clicks on 'Help and Info' in Vending
         * Machine.
         */
        public static final String VENDING_SUPPORT_URL = "vending_support_url";

        /**
         * Indicates if Vending Machine requires a SIM to be in the phone to allow a purchase.
         *
         * true = SIM is required
         * false = SIM is not required
         */
        public static final String VENDING_REQUIRE_SIM_FOR_PURCHASE =
                "vending_require_sim_for_purchase";

        /**
         * The current version id of the Vending Machine terms of service.
         */
        public static final String VENDING_TOS_VERSION = "vending_tos_version";

        /**
         * URL that points to the terms of service for Vending Machine.
         */
        public static final String VENDING_TOS_URL = "vending_tos_url";

        /**
         * Whether to use sierraqa instead of sierra tokens for the purchase flow in
         * Vending Machine.
         *
         * true = use sierraqa
         * false = use sierra (default)
         */
        public static final String VENDING_USE_CHECKOUT_QA_SERVICE =
                "vending_use_checkout_qa_service";
```
I located a few more interesting snippets - mainly about vending and requiring a sim card.

If there is anyone who is interested in testing some stuff out and has a dev/adp phone without a sim card - let me know. I'd be interested to see if a few patches I worked up would work :)