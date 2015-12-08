Feel free to ask questions! Either email, or I'm often in #libxbee on irc.freenode.net...

# NOTICE #
Google Code is shutting down. From now on, please go to GitHub for the libxbee source and support.
https://github.com/attie/libxbee3

## Overview ##
libxbee is a C/C++ library is to aid the use of Digi XBee radios running in API mode.

I have tried to keep flexibility to a maximum. By allowing connections to individual nodes to be created you don't have to address each packet, or filter through a list of packets to get at the one you're after. This is all done for you by libxbee.

To get a quick idea of how to use the library, see the samples in [the sample directory](http://code.google.com/p/libxbee/source/browse/?repo=libxbee-v3#git%2Fsample).

## Documentation ##
libxbee v3 is documented using the Unix man page system. For Windows users, and for general ease of access, these man pages have been converted to HTML, and are located in the Git repository, and are also hosted here: http://attie.github.io/libxbee3/ (from the gh-pages branch of the repo)

There is a [Getting Started Guide](http://attie.co.uk/libxbee/getting_started/) that covers many aspects of libxbee.

If you are running Linux or FreeBSD, you can easily install the man pages by cloning the Git repository, and following the installation steps below.

## Unix Installation & Use ##
libxbee v3 can be easily installed by following these steps:
  * make configure
  * make
  * sudo make install

This will configure, build and install the following key components:
  * a static library (libxbee.a)
  * a shared library (libxbee.so)
  * man page documentation

When linking your applications against libxbee, don't forget to add `-lxbee` to your compile line. On some systems you may also need to specify `-lpthread -lrt` as well.

## Example ##
Echo received data back using XBee Series 1 modules.
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <xbee.h>

void myCB(struct xbee *xbee, struct xbee_con *con, struct xbee_pkt **pkt, void **data) {
  if ((*pkt)->dataLen <= 0) return;
  
  /* tell the observer what we received */
  printf("rx: [%s]\n", (*pkt)->data);
  
  /* return the message to the other XBee */
  printf("tx: %d\n", xbee_connTx(con, NULL, (*pkt)->data, (*pkt)->dataLen));
}

int main(void) {
  struct xbee *xbee;
  struct xbee_con *con;
  struct xbee_conAddress address;
  
  /* setup libxbee, using a Series 1 XBee, on /dev/ttyUSB0, at 57600 baud */
  xbee_setup(&xbee, "xbee1", "/dev/ttyUSB0", 57600);
  
  /* setup a 64-bit address */
  memset(&address, 0, sizeof(address));
  address.addr64_enabled = 1;
  address.addr64[0] = 0x00;
  address.addr64[1] = 0x13;
  address.addr64[2] = 0xA2;
  address.addr64[3] = 0x00;
  address.addr64[4] = 0x40;
  address.addr64[5] = 0x08;
  address.addr64[6] = 0x18;
  address.addr64[7] = 0x26;
  
  /* create a 64-bit data connection with the address */
  xbee_conNew(xbee, &con, "64-bit Data", &address);
  
  /* setup a callback to keep both the system load and response time low */
  xbee_conCallbackSet(con, myCB, NULL);
  
  /* sleep for a minute! */
  sleep(60);
  
  /* close the connection */
  xbee_conEnd(con);
  
  /* shutdown libxbee */
  xbee_shutdown(xbee);
  
  return 0;
}
```