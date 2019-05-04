# Social Verification
At the moment the DigiAssets implementation supports asset verification by linking to a social media profile on one of the following platforms:
* Twitter
* Facebook
* Github

## Twitter

* You need the twitter handle of your twitter account **@your_twitter_handle**
* Add the following element under the asset metadata `verifications` key
```JSON
{	
	"social":{		
		"twitter":{			
			"username":"your_twitter_handle"
		},
	}
}
```

* After the asset is issued, grab the asset id (say it is `U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP`) and tweet the following text, making sure that the asset ID is used as a **hashtag**:

```Text
Verifying issuance of DigiAssets asset with ID #U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP
```

## Facebook
* Log in to your facebook account
* On the top right corner expand the menu and click on "Create Page"

![Creating a user page on Facebook](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/blob/master/images/fb_create_user_page.png)

* Select the appropriate category 
* Considering that this page should be dedicated to posting asset endorsment messages, you should probably name it properly and consider attaching an image and an appropriate description
* Now that you have created the page, click on the "Settings" button

![Finding the Facebook page settings button](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/blob/master/images/fb_page_settings_button.png)

* On the setting page, disable visitor posts

![Disabling visitor posts on a Facebook page](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/blob/master/images/fb_disable_visitors_posts.png)

* Grab the page ID from the url.
The URL looks something like:
<pre>
	https://www.facebook.com/Foobarbuzzquaxx-<b>705379359593101</b>/	
</pre>

The page ID is the number at the end of the URL, in the above case it is `705379359593101`

![Grab Facebook page id from URL](https://github.com/DigiAsset-Coins/DigiAsset-Coins-Protocol-Specification/blob/master/images/grabbing_fb_page_id.png)

* Add the following element under the asset metadata `verifications` key, for example
```JSON
{	
	"social":{		
		"facebook":{
			"page_id": 705379359593101					
		}
	}
}
```

* After the asset is issued, grab the asset id (say it is `U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP`) and **post on that page** the following text:
```Text
	Verifying issuance of DigiAssets asset with ID #U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP
```

## Github
* Create a **PUBLIC** gist for asset verifications
* grab the gist_id from the url, e.g. `https://gist.github.com/username/1d325dd9d1a74133bec3`
* Add the following element under the asset metadata `verifications` key
```JSON
{
	"social":{	
		"github":{			
			"gist_id":"1d325dd9d1a74133bec3"
		}
	}
}	
``` 
* For each asset that you want to verify (e.g. asset ids `U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP` and `LKUYHRCMbqUNgfNCGFnXv1AvB5Pv8Lkk2EjoF`) add a line to the gist like so:
```
...
Verifying issuance of DigiAssets asset with ID #U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP
Verifying issuance of DigiAssets asset with ID #LKUYHRCMbqUNgfNCGFnXv1AvB5Pv8Lkk2EjoF
...

```

## Domain Verification

* Place a text file on your server behind `httpS`
* Let's say the path to the file is `https://www.yourcompany.com/assets.txt`
* Add the following element under the asset metadata `verifications` key
```JSON
{	
	"domain":{
		"url":"https://www.yourcompany.com/assets.txt"		
	}
}
```
* If The file is not directly under the root of the website `https://www.yourcompany.com/path/to/file/filename.txt` you should use 
```JSON
{	
	"domain":{
		"url":"https://www.yourcompany.com/path/to/file/filename.txt"		
	}
}
```
For example:

```JSON
{	
	"domain":{
		"url":"https://www.example.com/digital_assets/assets.txt"		
	}
}
```

* For each asset that you want to verify (e.g. asset ids `U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP` and `LKUYHRCMbqUNgfNCGFnXv1AvB5Pv8Lkk2EjoF`) add the following two lines to the assets text file
```
...
U3uPyQeyNRafPy7popDfhZui8Hsw98B5XMUpP
LKUYHRCMbqUNgfNCGFnXv1AvB5Pv8Lkk2EjoF
...

```

You can check that the results of this check match what you get from [ssl_checker](https://www.sslshopper.com/ssl-checker.html), or [digicert](https://www.digicert.com/help/)
For example, surprisingly enough "https://www.target.com" doesn't pass the verification, and indeed, we [get the same result](https://www.sslshopper.com/ssl-checker.html#hostname=https://www.target.com) from ssl checker.