PicoAWSECommerceServiceClient
=============================

Pico Objective-C Client for the [Amazon Product Advertising API](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html)

## Note
1. This is only the proxy part of the PicoAWSECommerceServiceClient, you need to integrate with [Pico Runtime](https://github.com/bulldog2011/pico) before you can use this proxy, please follow instructions on Pico github site to add the Pico runtime library and the PicoAWSECommerceServiceClient in your iOS app, you may also start with the sample mentioned in Reference Sample section below. 
2. You need to fill in `AWSAccessKeyId` and `AWSSecureKeyId` in `AWSECommerceServiceClient` before your app can invoke Amazon Product Advertising service, and you must call `- (void)authenticateRequest:(NSString *)action` method in the `AWSECommerceServiceClient` to authenticate the request everytime you make a service call, see sample mentioned in Reference Sample section for details.
3. Besides proxy code, this site also hosts the [appledoc for the PicoAWSECommerceServiceClient](http://bulldog2011.github.com/PicoAWSECommerceServiceClient/). 

As per Amazon:
>You will not, without our express prior written approval, use any Product Advertising Content on or in connection with any site or application designed or intended for use with a mobile phone or other handheld device.

So please consult Amazon for permission before you can use its Product Advertising Content on any iOS devices.

##Example Usage
With this proxy and the generic Pico runtime library, Amazon Product Advertising API invocation on iOS platform is quite simple:

``` objective-c

        // start progress activity
        [self.view makeToastActivity];
        
        // get shared client
        AWSECommerceServiceClient *client = [AWSECommerceServiceClient sharedClient];
        client.debug = YES;
        
        // build request, see details here:
        ItemSearch *request = [[[ItemSearch alloc] init] autorelease];
        request.associateTag = @"tag"; // seems any tag is ok
        request.shared = [[[ItemSearchRequest alloc] init] autorelease];
        request.shared.searchIndex = @"Books";
        request.shared.responseGroup = [NSMutableArray arrayWithObjects:@"Images", @"Small", nil];
        ItemSearchRequest *itemSearchRequest = [[[ItemSearchRequest alloc] init] autorelease];
        itemSearchRequest.title = _searchText.text;
        request.request = [NSMutableArray arrayWithObject:itemSearchRequest];
        
        // authenticate the request
        // http://docs.aws.amazon.com/AWSECommerceService/latest/DG/NotUsingWSSecurity.html
        [client authenticateRequest:@"ItemSearch"];
        [client itemSearch:request success:^(ItemSearchResponse *responseObject) {
            // stop progress activity
            [self.view hideToastActivity];
            
            // success handling logic
            if (responseObject.items.count > 0) {
                Items *items = [responseObject.items objectAtIndex:0];
                if (items.item.count > 0) {
                    Item *item = [items.item objectAtIndex:0];
                    
                    // start image downloading progress activity
                    [self.view makeToastActivity];
                    // get gallery image
                    NSURL *imageURL = [NSURL URLWithString:item.smallImage.url];
                    NSData *imageData = [NSData dataWithContentsOfURL:imageURL];
                    // stop progress activity
                    [self.view hideToastActivity];
                    
                    UIImage *image = [UIImage imageWithData:imageData];
                    [self.view makeToast:item.itemAttributes.title duration:3.0 position:@"center" title:@"Success" image:image];
                } else {
                    // no result
                    [self.view makeToast:@"No result" duration:3.0 position:@"center"];
                }
                
            } else {
                // no result
                [self.view makeToast:@"No result" duration:3.0 position:@"center"];
            }
        } failure:^(NSError *error, id<PicoBindable> soapFault) {
            // stop progress activity
            [self.view hideToastActivity];
            
            // error handling logic
            if (error) { // http or parsing error
                [self.view makeToast:[error localizedDescription] duration:3.0 position:@"center" title:@"Error"];
            } else if (soapFault) {
                SOAP11Fault *soap11Fault = (SOAP11Fault *)soapFault;
                [self.view makeToast:soap11Fault.faultstring duration:3.0 position:@"center" title:@"SOAP Fault"];
            }
        }];

        
```

## Reference Sample 

* [ASWECommerce](https://github.com/bulldog2011/pico/tree/master/Examples/AWSECommerce) - Hello world like sample using [Amazon Product Advertising API](https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html) SOAP call.
* [AWSECDemoApp](https://github.com/bulldog2011/pico/tree/master/Examples/AWSECDemoApp) - Sample Amazon Book search and purchase app using Amazon Product Advertising API.

##Docs
1. [Wsdl Driven Development on iOS - the Big Picture](http://bulldog2011.github.com/blog/2013/03/25/wsdl-driven-development-on-ios-the-big-picture/)
2. [Pico Tutorial 5 - Hello Amazon Product Advertising API](http://bulldog2011.github.com/blog/2013/03/31/pico-tutoiral-5-hello-amazon-product-advertising-api/)


##Copyright and License
(The MIT License)

Copyright (c) 2013 Leansoft Technology <51startup@sina.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE. 

