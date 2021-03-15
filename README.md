# Install
Download the training zip and unzip.

# Mac/Linux
From the newly unzipped directory run:

`./bin/janusgraph start`

After it starts, run ./bin/gremlin.sh and the following commands:

```
:remote connect tinkerpop.server conf/remote.yaml session
:remote console
g.V().count()
```
# Windows
Make sure Java 8 is installed, and unzip the file and then navigate into the directory. Then:
1. Run gremlin-server.bat
1. Open a new Gremlin shell: gremlin.bat

Then run the following commands:

```
:remote connect tinkerpop.server conf/remote.yaml session
:remote console
g.V().count()
```
