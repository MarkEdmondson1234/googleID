# googleID

status - *deprecated* - Google+ API is not valid anymore.  Use googleSignIn() from `googleAuthR` instead. 

[![Travis-CI Build Status](https://travis-ci.org/MarkEdmondson1234/googleID.svg?branch=master)](https://travis-ci.org/MarkEdmondson1234/googleID)

Authentication and identifying Google users using Google+ API

## Example

Activate [Google+ API here](https://console.developers.google.com/apis/api/plus/overview)

Download your Google Project's client JSON as detailed in [googleAuthR help](http://code.markedmondson.me/googleAuthR/articles/google-authentication-types.html#setting-the-client-via-google-cloud-client-json)

```r
library(googleAuthR)
library(googleID)

## set your client ID/secret and scopes
gar_set_client("location_of_client.json",
               scopes = c("https://www.googleapis.com/auth/userinfo.email",
                          "https://www.googleapis.com/auth/userinfo.profile"))

# or if you have downloaded service auth JSON, gar_service_auth()
gar_auth()

## default is user logged in
user <- get_user_info()

> str(user)
List of 24
 $ kind          : chr "plus#person"
 $ etag          : chr "\"4OZ_Kt6ujOh1jaML_U6RM6APqoE/UYyiZzYR_iS62bMpKv92P1iPcZ0\""
 $ occupation    : chr "Google Developer Expert - Google Analytics.  RStudio Advocate."
 $ skills        : chr "Google Analytics, Adobe Marketing Cloud, R, Python, SEO, Google Cloud, Statistics, Music"
 $ gender        : chr "male"
 $ emails        :'data.frame':	1 obs. of  2 variables:
  ..$ value: chr "x@xxxx.xx"
  ..$ type : chr "account"
 $ urls          :'data.frame':	5 obs. of  3 variables:
  ..$ value: chr [1:5] "http://twitter.com/holomarked" "http://www.linkedin.com/in/markpeteredmondson" "https://github.com/MarkEdmondson1234" "http://stackoverflow.com/users/3878063/marked" ...
  ..$ type : chr [1:5] "otherProfile" "otherProfile" "otherProfile" "otherProfile" ...
  ..$ label: chr [1:5] "Twitter" "Mark Edmondson LinkedIn" "Github" "StackOverflow" ...
 $ objectType    : chr "person"
 $ id            : chr "115064340704113209584"
 $ displayName   : chr "Mark Edmondson"
 $ name          :List of 2
  ..$ familyName: chr "Edmondson"
  ..$ givenName : chr "Mark"
 $ tagline       : chr "Taking part in the evolution of the Multivac"
 $ braggingRights: chr "RStudio Advocate.  Google Developer Expert - Google Analytics.  Adobe Certified Expert - Adobe Analytics."
 $ aboutMe       : chr "Taking part in the evolution of the multivac<br />"
 $ url           : chr "https://plus.google.com/+MarkEdmondsonAtHome"
 $ image         :List of 2
  ..$ url      : chr "https://lh5.googleusercontent.com/-OVYhmgQg3lg/AAAAAAAAAAI/AAAAAAAAAP0/1qVcsNpSXlQ/photo.jpg?sz=50"
  ..$ isDefault: logi FALSE
 $ organizations :'data.frame':	1 obs. of  4 variables:
  ..$ name   : chr "King's College London"
  ..$ title  : chr "Physics"
  ..$ type   : chr "school"
  ..$ primary: logi FALSE
 $ placesLived   :'data.frame':	3 obs. of  1 variable:
  ..$ value: chr [1:3] "Copenhagen" "London" "Falmouth, UK"
 $ isPlusUser    : logi TRUE
 $ language      : chr "en_GB"
 $ circledByCount: int 410
 $ verified      : logi FALSE
 $ cover         :List of 3
  ..$ layout    : chr "banner"
  ..$ coverPhoto:List of 3
  .. ..$ url   : chr "https://lh3.googleusercontent.com/HIq8ABRcUKBs5AYBg0tiqoJifjv3A08MWCU3u79hBWhXjvBv1iilf94_YIjRSH4VlWZH=s630"
  .. ..$ height: int 183
  .. ..$ width : int 940
  ..$ coverInfo :List of 2
  .. ..$ topImageOffset : int 0
  .. ..$ leftImageOffset: int 0
 $ domain        : chr "markedmondson.me"

```

## Use within Shiny

```r
library(shiny)
library(googleAuthR)
library(googleID)

# set scopes
gar_set_client(scopes = c("https://www.googleapis.com/auth/userinfo.email",
                                       "https://www.googleapis.com/auth/userinfo.profile"))

## set your web app client ID/secret and scopes
options(googleAuthR.webapp.client_id = getOption("googleAuthR.client_id"),
        googleAuthR.webapp.client_secret = getOption("googleAuthR.client_secret"))

ui <- shinyUI(fluidPage(
  
  googleAuthUI("example1"),
  p("Logged in as: ", textOutput("user_name"))
  
))


server <- shinyServer(function(input, output, session) {
  
  access_token <- callModule(googleAuth, "example1")
  
  ## to use in a shiny app:
  user_details <- reactive({
    validate(
      need(access_token(), "Authenticate")
    )
    
    with_shiny(get_user_info, shiny_access_token = access_token())
    
  })
  
  output$user_name <- renderText({
    validate(
      need(user_details(), "getting user details")
    )
    
    user_details()$displayName
    
  })
  
  
})

# Run the application 
shinyApp(ui = ui, server = server)

```

## Shiny/RMarkdown via JavaScript auth plugin

Alternatively, you can also login via javascript in RMarkdown documents, but this requires the dev version of googleAuthR.  
A demo and how to of this is available in this RMarkdown document: https://mark.shinyapps.io/googleAuthRMarkdown/

## Whitelist

The function `whitelist()` can be used to return TRUE or FALSE is the user is on a supplied list.

You can supply the whitelist as a character vector, which could be imported from a file or similar.

To use, pass the user object to the function with the whitelist like this:

```r
library(googleAuthR)
library(googleID)

## set your client ID/secret and scopes
gar_set_client("location_of_client.json",
               scopes = c("https://www.googleapis.com/auth/userinfo.email",
                          "https://www.googleapis.com/auth/userinfo.profile"))

gar_auth()

## default is user logged in
user <- get_user_info()

the_list <- whitelist(user, c("your@email.com", "another@email.com", "yet@anotheremail.com"))

if(the_list){
  message("You are on the list.")
} else {
  message("If you're not on the list, you're not getting in.")
}
```
