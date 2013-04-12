---
layout: post
title: "Why HTTP ETag Header May Cause Your Downloading Apps On Android Failed"
description: ""
category: 
tags: [android]
author:  <a href="https://github.com/socrateslee">Chun Li</a>
---
{% include JB/setup %}

We provide directly apk download from our servers for [Applood](http://appflood.com) SDK in case there are some areas have difficulty to download apps from Google Play.

Yesterday on of our engineer found that an apk could not be correctly downloaded from CDN networks with dolphin browser or default Android browser. After a few test, we located the issue, which are not caused by browsers, neither the CDN networks or the source servers, the problem is with somehow with the default Android Download Manager. You may reproduce the error by download a small file repeatly with a fast networks, and you may witness a download is reported failed right after the download completed.

We connect our testing phone with logcat, and found there is a "Trying to resume a download that can't be resumed" error raised, or [this error code](http://developer.android.com/reference/android/app/DownloadManager.html#ERROR_CANNOT_RESUME) in Andorid document. After google it, we also found [this issue](https://code.google.com/p/android/issues/detail?id=18462) reported exactly the same error with us, but seems there is no work around about this except use an alternate download manager.

So we had a look at [the source code of the Android download manager](https://android.googlesource.com/platform/packages/providers/DownloadProvider/), and we found the code block which raise the error in _DownloadThread.java_.

    /**                                                                                                                             
     * Prepare the destination file to receive data.  If the file already exists, we'll set up                                      
     * appropriately for resumption.                                                                                                
     */
    private void setupDestinationFile(State state, InnerState innerState)
            throws StopRequestException {
        if (!TextUtils.isEmpty(state.mFilename)) { // only true if we've already run a thread for this download  
    ...
        // We're resuming a download that got interrupted                                                                       
            File f = new File(state.mFilename);
            if (f.exists()) {
    ...
                } else if (mInfo.mETag == null && !mInfo.mNoIntegrity) {
                // This should've been caught upon failure                                                                      
                if (Constants.LOGVV) {
                    Log.d(TAG, "setupDestinationFile() unable to resume download, deleting "
                            + state.mFilename);
                }
                f.delete();
                throw new StopRequestException(Downloads.Impl.STATUS_CANNOT_RESUME,
                        "Trying to resume a download that can't be resumed");
            } else {
    ...

Here it comes the ETag header. The default Android download manager use ETag header to determine whether the downloaded content is the same resource. If you have two threads downloading the same resource, the download manager will check whether they have the same ETag, an error will be raised if no Etag is provided(Yes, we have no ETag header). The default Android browser will send an GET request to the server and get the content reponsed, if the content is an apk the browser will send an intent to the download manager and let it download it again. The beheavior of the browser may explain why there are two concurrent download threads, and first thread may not finisesh when the new thread of the download manager runs.

So we add a ETag header on our server, and it sovle the problem.