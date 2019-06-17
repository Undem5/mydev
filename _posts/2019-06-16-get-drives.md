---
category: Drives
path: '/drives'
title: 'Get drives'
type: 'GET'

layout: nil
---

### Windows

Windows give us a way to access to our disk drives thanks to `unsigned long _getdrives( void );` from `direct.h`
It returns a bitmask where the least-significant bit is the first drive and then you need to shift the bit to the right the get the second hard drive.


### Code
```
#include <iostream>
#include <Windows.h>
#include <direct.h>


int main(int argc, char *argv[])
{
	ULONG DriveMask = _getdrives();
	char mydrives[] = " A: ";

	if (DriveMask == 0) {
		printf("_getdrives() failed, error code: %d", GetLastError());
	}
	else {
		printf("Logical drives:\n");
		while (DriveMask) {
			if (DriveMask & 1)
				printf(mydrives);
			++mydrives[1];
			DriveMask >>= 1;
		}
		printf("\n");
	}

	return 0;

}
```

### Output

```
Logical Drives:
  C: D: E:
```
