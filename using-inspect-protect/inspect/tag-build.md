# Tagging a Build Based on Analysis Results

A build in your CI-CD pipeline can be automatically tagged based on ShiftLeft Inspect's code analysis results. You can specify a build rule for your organization to tag a build, if 

* The number of vulnerabilities crosses a specific threshold.

* At least one occurrence of a specific type of vulnerability is present in the source code.

Once you have tagged a build, the build is shown in the Version drop-down menu (you may need to refresh first), allowing you to view the analysis results for that specific build.

**The process of specifying a build rule is**:

1. [Create a filter that defines the tag criteria](filter-results.md#creating-a-filter).
2. [Save the filter](filter-results.md#saving-a-filter).
3. In the Filter Saver dialog, check **Build Rule for Org**.

     ![Tag Build Rule](img/tag-build-rule.jpg)

4. Use the following bash script, which invokes ShiftLeft's public API. 
   The script shows 1 for success if the ShiftLeft Inspect results don’t have any vulnerabilities that match the build rule criteria. The script shows 0 for failure, if the build has vulnerabilities that match the build rule criteria.
 
```
#!/bin/sh

set -e

#Insert your public token here, as provided by ShiftLeft UI (user profile view with feature flag ?publicApiToken=enable)
PUBLIC_TOKEN=`cat token.txt`

#Insert your organization ID as provided in the ShiftLeft UI
ORGID='-----------------------------------'
#Insert your application ID as provided in the ShiftLeft UI
APPID='----------'

#We're assuming this is the "tag_key=tag_value" use case "branch=branch_name"
TAG_KEY='branch'
TAG_VALUE=$1

SL_PROTO='https'
SL_HOST='www.shiftleft.io'
SL_URL="$SL_PROTO://$SL_HOST/api/v3/public/org/$ORGID/app/$APPID/tag/$TAG_KEY/$TAG_VALUE/build"
BEARER='Authorization: Bearer '$PUBLIC_TOKEN
BUILD_RESULT=$(curl --fail --show-error -X GET \
   $SL_URL \
  -H 'Accept: */*' \
  -H "$BEARER" \
  -H 'Cache-Control: no-cache' \
  -H 'Connection: keep-alive' \
  -H "Host: $SL_HOST" \
  -H 'accept-encoding: text/plain, deflate' \
  -H 'cache-control: no-cache' \
  -H 'cookie: Cookie_3=value' \
  -s -b Cookie_3=value)

echo $BUILD_RESULT

SUCCESS=$(echo $BUILD_RESULT | grep "success")
if [ -z "$SUCCESS" ]; then
    exit 1;
fi
exit 0
```
