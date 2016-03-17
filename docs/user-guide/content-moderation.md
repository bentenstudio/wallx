## Moderation Mode

Moderation mode means that all submissions will need to be approved before showing in app. 

To turn this on, open Config.java file and change **MODERATION_MODE** to **true**. This value is **false** by default.

## Photo Approval
By default, any photo that was uploaded by users will have this value as false. It is only true when photos are uploaded by Dashboard or it was marked as true later in the Dashboard >> Moderation

To approve a photo, login to Dashboard and browse to Moderation section. Pick the photo that needs to be moderated and click "Approve" button. 

Or you can manually change `isApproved` value to `true` in any wallpaper object.

## Flag Threshold
Flag threshold: a value which decides if a photo is appropriate or not. This value is increased whenever a logged-in user choose "Flag inappropriate" option in app. Any approved photo will be displayed no matter how high this value is.