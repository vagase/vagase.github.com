---
layout: post
title: Objective-C Singleton template
categories:
- Objective C
- sharedInstance
- Singleton
- Template
status: publish
type: post
published: true
comments: true
---
原文地址：[http://blog.mugunthkumar.com/coding/objective-c-singleton-template-for-xcode-4/](http://blog.mugunthkumar.com/coding/objective-c-singleton-template-for-xcode-4/)

####\___FILEBASENAME___.h
{% codeblock lang:objc %}
#import <foundation /Foundation.h>

@interface ___FILEBASENAMEASIDENTIFIER___ : NSObject {

}

+ (___FILEBASENAMEASIDENTIFIER___*) sharedInstance;

@end
{% endcodeblock %}

<!-- More -->

####\___FILEBASENAME___.m
{% codeblock lang:objc %}
#import "___FILEBASENAME___.h"

static ___FILEBASENAMEASIDENTIFIER___ *_instance;
@implementation ___FILEBASENAMEASIDENTIFIER___

#pragma mark -
#pragma mark Singleton Methods

+ (___FILEBASENAMEASIDENTIFIER___*)sharedInstance{
	if(_instance == nil){
		@synchronized(self) {		
	        if (_instance == nil) {
	            _instance = [[self alloc] init];
	            
	            // Allocate/initialize any member variables of the singleton class here
	            // example
				//_instance.member = @"";
	        }
	    }
	}
    return _instance;
}

+ (id)allocWithZone:(NSZone *)zone{	
	if(_instance == nil){
		@synchronized(self) {
        	if (_instance == nil) {
	            _instance = [super allocWithZone:zone];			
    	        return _instance;  // assignment and return on first allocation
        	}
	    }
	}
    return nil; //on subsequent allocation attempts return nil	
}

- (id)copyWithZone:(NSZone *)zone{
    return self;	
}

- (id)retain{	
    return self;	
}

- (unsigned)retainCount{
    return UINT_MAX;  //denotes an object that cannot be released
}

- (void)release{
    //do nothing
}

- (id)autorelease{
    return self;	
}

#pragma mark -
#pragma mark Custom Methods

// Add your custom methods here

@end
{% endcodeblock %}