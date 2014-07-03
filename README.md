## Apple TV2 Development

### Requirements
- [Jailbroken](http://blog.firecore.com/11550) AppleTV running iOS 6  (Version 5.3)
- SSH Access (Remember to change your root password)

### Resources

- [HelloWorldPlugin for iOS 6 using Theos/Logos](https://github.com/lechium/HelloWorld_ATVBlack6)
- [Theos installation instructions](http://www.reddit.com/r/jailbreak/comments/20nxy8/flex_to_mobilesubstrate_a_beginners_guide_to/), ignore the rest.
- [libsubstrate.dylib and substrate.h](https://github.com/kokoabim/iOSOpenDev) here.
- [Logos Documentation](http://iphonedevwiki.net/index.php/Logos) for writing Jailbreak and late binding code
- [Theos Build System Documentation](https://github.com/theiostream/theos-ref)
- [NSHipster Article on BackRow](http://www.nshipster.com/backrow)
- [AppleTV Private Headers](https://github.com/lechium/appletv-private-headers)
- [Excellent Apple TV project example](https://github.com/alloy/UitzendingGemist-ATV2) but for iOS 5. Still a great reference to roughly understand the BackRow Classes

###Writing ARC in an MRC world

Due to the fact that iOS 6 has all the `BackRow` classes embedded into the `AppleTV.app`, we have to use runtime subclassing, associated objects, etc. which is what `theos/logos` helps us do.  The `theos` build system compiles using GCC, there by forcing us to use MRC. 

In order to work around this issue, we can separately compile our application specific code as a static library with ARC enabled and link to the plugin at compile time. Instantiating these objects can be done in the same manner as we do with creating objects at runtime: `objc_getClass`.

-------------------------------------------------------------------------------------

## Original Documentation

AppleTV HelloWorld plugin for AppleTV 2, pretty self explanatory.

**iOS 6 notes**

Going to cover some basic notes that are very important for porting your plugins for iOS 6.

In iOS 6 Apple has removed the AppleTV.framework and merged its classes into AppleTV.app. There are some incredibly annoying side effects that result from this, making our lives much more difficult.

The only way to inherit from any of these classes is to dynamically create them at runtime, the easiest way to do this, is moving back to doing things with theos/logos

An unfortunate side effect of doing it through logos is there is no support for adding ivars, so everything needs to be done through associated objects, this can make large re-writes cumbersome, however, since associated objects handle memory management for you, and adding ivars at runtime would be a huge pain, its the best way to go.

Another change is BRControls no longer inherit directly from CALayer, and now are UIView the control hierarchy has totally been turned on its head. no more _dumpControlTree and no more iterating through [BRControl controls] to get at a specific UI control, they are subviews now. same idea still essentially works though. i have to do this a lot when i need to iterate through controls.

NSArray *controlArray = nil;
if ([self respondsToSelector:@selector(controls)]) { controlArray = [self controls]; } else { controlArray = [self subviews]; }

BRImageAndSyncingPreviewController is gone so we replace it with something that is NOT comparable: BRIconPreviewController.

**main menu image:** all of them are stored inside AppleTV.app now, so you need a copy of the icon as AppIcon.png inside your .frappliance bundle, and you need to add Applications/AppleTV.app/ with your identifier @720p.png ie: "com.apple.frontrow.appliance.hw@720p.png"

**theos modification notes:**

So this is a slightly outdated version of theos (could've swore i updated it to the latest, guess not) so some of the constants in the make file have since been renamed again in more recent versions.

the main modifications made is to build in debug mode (export DEBUG=1), this builds the plugin with all the symbols for debugging and crash symbolication intact
from here in the after-HW-stage we copy the unstripped binary into our root folder, generate a dSYM file, strip and re-sign the binary so its properly stripped for packaging and releasing.

also needed to make a modification to comment out the addition of debug or whatever in theos/bin/package_version.sh commenting out lines 65-67, without doing this it adds +DEBUG or something along those lines to the version number in the control file.

if [[ ! -z "$EXTRAVERS" ]]; then
	extra_part="+$EXTRAVERS"
fi

without all these extra modifications debugging crashes is insanely difficult.
