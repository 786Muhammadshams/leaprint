This repository is used to build the LEAPrint Container.

# Step 0: Run a linux container for development/builds

You'll need to build LEAPrint in a linux environment. You can try `rake dev`,
but I haven't tested this yet. If that doesn't work, you can use `./dev.sh`,
but you may have to adapt this for podman if you don't have docker installed.

# Step 1: Build the server exe

Attach to the linux build/dev environment, then you can go to the `/app`
directory since it's mounted in this container from the local file sytem.
Changes on either side will be reflected on the other.

In the `Server/Serializer` directory, run `make`. This builds `Serializer.a`
which is used in the next step. It's not necessary to build `dr`'s source files
separately since it's built in `dxx` below.

In the `Server/` directory, run the `dxx` script. The dependencies (Poco,
boost, cairo, pixman) must already be installed. This builds
LEAPrintServer.cpp to make `a.out`. You can now run this executable within the
linux environment.

# Step 2: Test the server exe

Use `curl localhost:9090` within this container and it should show you a
message "LEA Print is up".

In the `Server/` directory, run the `request` script. This POSTs a JSON file to
the server you ran above. This will write a Rendered.pdf. Open this to make
sure it worked. You should see a drive order.

You'll have to do both these within the linux environment, or else you won't
reach it, unless you change the `rake dev` to forward port 9090 to the
container.

# Step 3: Build and upload the image

In the root directory, run `rake builder`. This uses podman to fire up a
container where buildah is running.

Next, run `rake copy`. This copies the server exe and other needed files to
the running buildah container.

Then run `rake image`. This builds a cri-o image with the LEAPrint server and
its dependencies.

Finally run `rake push`. You'll need to copy the image hash printed from the
previous step. Also this won't work if you don't have permission to write
images to our github container repository. Ask Mark if you want to try this.
