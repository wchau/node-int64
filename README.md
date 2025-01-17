# ![image](https://user-images.githubusercontent.com/164050/52737878-42886600-2f82-11e9-81de-c750db7b4ac3.png) This package is not actively maintained

`int64` was a workaround JS' lack of support for Int64 data types.  Somewhat to my surprise, people have found it useful.  However, it looks like [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt)s will soon be a formally accepted feature, obsoleting the need for this package.

If a trustworthy organization or individual would like to take over as maintainer, I'm happy to hand this off.  Please contact me at [robert@broofa.com](mailto:robert@broofa.com) if you're interested. Otherwise I'll be npm-deprecating this at some future date.

# int64

This is a fork that works with Google Apps Script through https://github.com/mahaker/esbuild-gas-plugin

JavaScript Numbers are represented as [IEEE 754 double-precision floats](http://steve.hollasch.net/cgindex/coding/ieeefloat.html).  Unfortunately, this means they lose integer precision for values beyond +/- 2^^53.  For projects that need to accurately handle 64-bit ints, such as [node-thrift](https://github.com/wadey/node-thrift), a performant, Number-like class is needed.  Int64 is that class.

Int64 instances look and feel much like JS-native Numbers.  By way of example ...
```js
// First, let's illustrate the problem ...
> (0x123456789).toString(16)
'123456789' // <- what we expect.
> (0x123456789abcdef0).toString(16)
'123456789abcdf00' // <- Ugh!  JS doesn't do big ints. :(

// So let's create a couple Int64s using the above values ...

// Require, of course
> Int64 = require('node-int64')

// x's value is what we expect (the decimal value of 0x123456789)
> x = new Int64(0x123456789)
[Int64 value:4886718345 octets:00 00 00 01 23 45 67 89]

// y's value is Infinity because it's outside the range of integer
// precision.  But that's okay - it's still useful because it's internal
// representation (octets) is what we passed in
> y = new Int64('123456789abcdef0')
[Int64 value:Infinity octets:12 34 56 78 9a bc de f0]

// Let's do some math.  Int64's behave like Numbers.  (Sorry, Int64 isn't
// for doing 64-bit integer arithmetic (yet) - it's just for carrying
// around int64 values
> x + 1
4886718346
> y + 1
Infinity

// Int64 string operations ...
> 'value: ' + x
'value: 4886718345'
> 'value: ' + y
'value: Infinity'
> x.toString(2)
'100100011010001010110011110001001'
> y.toString(2)
'Infinity'

// Use JS's isFinite() method to see if the Int64 value is in the
// integer-precise range of JS values
> isFinite(x)
true
> isFinite(y)
false

// Get an octet string representation.  (Yay, y is what we put in!)
> x.toOctetString()
'0000000123456789'
> y.toOctetString()
'123456789abcdef0'

// Finally, some other ways to create Int64s ...

// Pass hi/lo words
> new Int64(0x12345678, 0x9abcdef0)
[Int64 value:Infinity octets:12 34 56 78 9a bc de f0]

// Pass a Buffer
> new Int64(new Buffer([0x12, 0x34, 0x56, 0x78, 0x9a, 0xbc, 0xde, 0xf0]))
[Int64 value:Infinity octets:12 34 56 78 9a bc de f0]

// Pass a Buffer and offset
> new Int64(new Buffer([0,0,0,0,0x12, 0x34, 0x56, 0x78, 0x9a, 0xbc, 0xde, 0xf0]), 4)
[Int64 value:Infinity octets:12 34 56 78 9a bc de f0]

// Pull out into a buffer
> new Int64(new Buffer([0x12, 0x34, 0x56, 0x78, 0x9a, 0xbc, 0xde, 0xf0])).toBuffer()
<Buffer 12 34 56 78 9a bc de f0>

// Or copy into an existing one (at an offset)
> var buf = new Buffer(1024);
> new Int64(new Buffer([0x12, 0x34, 0x56, 0x78, 0x9a, 0xbc, 0xde, 0xf0])).copy(buf, 512);
```
