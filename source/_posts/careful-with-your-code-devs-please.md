---
title: Careful with your code devs! Please!
tags:
  - .bak
  - android
  - code
  - devs
  - doh
  - double-check
  - g1
  - LicenseActivity.java.bak
  - licensing
  - mistake
  - reverse engineering
  - security
  - snippet
url: careful-with-your-code-devs-please.html
id: 161
categories:
  - android
date: 2009-01-05 17:07:53
---

![Keep your code secure!](http://173.230.150.16/blog/wp-content/uploads/2009/01/strictly_confidential-230x300.jpg "Keep your code secure!")
I was looking around at some applications on the Android Market today and was trying to look at how some of the applications well, did what they did. I was also interested in looking at how they did they're own licensing schemes, anyways. Hopefully all the developers know this - but maybe some of this, surprisingly, don't know this. A .apk file is merely a .jar file - meaning it is a **zipped** up. This means it can **easily be unzipped**. So before releasing things, you should _really_ double check your .apk file and make sure it contrains _only_ what you want. This would normally include a `/res` folder, `/META` folder and you main directory that has `AndroidManifest.xml`, `classes.dex` and `resources.arsc`.
![Source code is usually NOT given to users, right?](http://173.230.150.16/blog/wp-content/uploads/2009/01/confidential_stamp1-300x250.jpg "Source code is usually NOT given to users, right?")
You should definitely check if it contains extra files like `LicenseActivity.java.bak`;
```
	@Override
	protected void onResume() {
		super.onResume();

		checkClipboard();
	}


	/**
	 *
	 */
	private void checkClipboard() {
		// If someone copied a license, paste it now:
		ClipboardManager clippy = (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);
		if (clippy.hasText()) {
			String clip = clippy.getText().toString();
			clip = clip.trim();
			Log.i(TAG, "Clipboard: " + clip);
			if (clip.length() == 19
					&& clip.substring(4,5).equals("-")
					&& clip.substring(9,10).equals("-")
					&& clip.substring(14,15).equals("-")) {
				// This is a license:
				setLicenseText(clip);

				Toast.makeText(this, getString(RD.string.license_from_clipboard), Toast.LENGTH_LONG).show();
			}
		}
	}


	/**
	 * @param licensekey
	 */
	private void setLicenseText(String licensekey) {
		if (licensekey != null && licensekey.length() == 19) {

			mLic1.setText(licensekey.substring(0, 4));
			mLic2.setText(licensekey.substring(5, 9));
			mLic3.setText(licensekey.substring(10, 14));
			mLic4.setText(licensekey.substring(15, 19));
		}
	}

	protected void requestLicense(String targetUri) {
		Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse(targetUri
				+"?ctmcode=" + mCtmCode + "&appcode=" + mAppCode + "liccode="
				+ mLicCode));
		startActivity(intent);

	}

	protected void storeLicenseAndFinish() {
		Editor editor = PreferenceManager.getDefaultSharedPreferences(this)
				.edit();
		StringBuilder sb = new StringBuilder();
		sb.append(mLic1.getText());
		sb.append('-');
		sb.append(mLic2.getText());
		sb.append('-');
		sb.append(mLic3.getText());
		sb.append('-');
		sb.append(mLic4.getText());

		if (LicenseChecker.checkLicense(this, sb.toString())) {
			editor.putString(LicenseChecker.PREFS_LICENSE_KEY, sb.toString());
			editor.commit();

			((LicensedApplication) getApplication()).newLicense();
			finish();
		} else {
			Toast.makeText(this, getString(RD.string.invalid_license), Toast.LENGTH_LONG).show();
		}

	}

	private class TextChangedWatcher implements TextWatcher {

		TextView mPrevTextView;
		TextView mNextTextView;

		public TextChangedWatcher(TextView prevTextView, TextView nextTextView) {
			mPrevTextView = prevTextView;
			mNextTextView = nextTextView;
		}

		public void afterTextChanged(Editable s) {
			// Jump focus to next when field is full
			if (s.toString().length() == 4 && mNextTextView != null) {
				mNextTextView.requestFocus();
			}

			// Jump focus to previous when last character has been deleted
			if (s.toString().length() == 0 && mPrevTextView != null) {
				mPrevTextView.requestFocus();
			}
		}

		public void beforeTextChanged(CharSequence s, int start, int count,
				int after) {
		}

		public void onTextChanged(CharSequence s, int start, int before,
				int count) {
		}
	}


	@Override
	protected Dialog onCreateDialog(int id) {

		switch (id) {
		case DIALOG_ID_MARKET_WARNING:
			return new AlertDialog.Builder(LicenseActivity.this).setIcon(
					android.R.drawable.ic_dialog_alert).setTitle(RD.string.alert).setMessage(
					RD.string.dialog_market_warning).setPositiveButton(
					android.R.string.ok, new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog,
								int whichButton) {
							requestLicense(LICENSE_MARKET_URL);
							finish();
						}
					}).setNegativeButton(android.R.string.cancel,
					new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog,
								int whichButton) {
						}
					}).create();
			//TODO: fix me. what should be in those strings?
		case DIALOG_ID_HANDANGO_WARNING:
			return new AlertDialog.Builder(LicenseActivity.this).setIcon(
					android.R.drawable.ic_dialog_alert).setTitle(RD.string.alert).setMessage(
					RD.string.dialog_market_warning).setPositiveButton(
					android.R.string.ok, new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog,
								int whichButton) {
							requestLicense(LICENSE_HANDANGO_URL);
							finish();
						}
					}).setNegativeButton(android.R.string.cancel,
					new DialogInterface.OnClickListener() {
						public void onClick(DialogInterface dialog,
								int whichButton) {
						}
					}).create();

		}
		return null;
	}

}
```
Or more files... Which might essentially give your "paid" application out for free, and it's source code essentially open source. While it's a "nice" thing that you've essentially open sourced your application. I'm sure it wasn't your intent to do so, and distribute it on the Android Market and your website.

Hopefully no one else other than the developer (who has been contacted) has to learn this lesson... Shesh! There was also a few other (and large fully working) .java.bak files that where include, above is just a "licensing snippet" I was looking over. Though I've only pasted a small portion that doesn't include any indication on exactly which application it is.