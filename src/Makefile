APPNAME=redeclipse
APPCLIENT=$(APPNAME)
APPSERVER=$(APPNAME)_server

#CXXFLAGS= -ggdb3
CXXFLAGS?= -O3 -fomit-frame-pointer -ffast-math
override CXXFLAGS+= -Wall -fsigned-char -fno-exceptions -fno-rtti

PLATFORM= $(shell $(CC) -dumpmachine)

RMFLAGS= -fv

ifeq (,$(PLATFORM_BIN))
ifneq (,$(findstring arm,$(PLATFORM)))
PLATFORM_BIN=arm
else
ifneq (,$(findstring 64,$(PLATFORM)))
PLATFORM_BIN=amd64
else
PLATFORM_BIN=x86
endif
endif
endif

ifeq (,$(INSTDIR))
INSTDIR=../bin/$(PLATFORM_BIN)
endif

TOOLSET_PREFIX=
ifneq (,$(findstring cross,$(PLATFORM)))
ifneq (,$(findstring 64,$(PLATFORM)))
TOOLSET_PREFIX=x86_64-w64-mingw32-
else
TOOLSET_PREFIX=i686-w64-mingw32-
endif
CXX=g++
CC=gcc
endif

CXX_TEMP:=$(CXX)
CC_TEMP:=$(CC)
override CXX=$(TOOLSET_PREFIX)$(CXX_TEMP)
override CC=$(TOOLSET_PREFIX)$(CC_TEMP)

INCLUDES= -I. -Ishared -Iengine -Igame -Ienet/include -Isupport

STRIP=
ifeq (,$(findstring -g,$(CXXFLAGS)))
ifeq (,$(findstring -pg,$(CXXFLAGS)))
	STRIP=strip
endif
endif
STRIP_TEMP:=$(STRIP)
override STRIP=$(TOOLSET_PREFIX)$(STRIP_TEMP)

MV=mv
CP=cp
MKDIR=mkdir -p

ifneq (,$(findstring mingw,$(PLATFORM)))
APPMODIFIER=_windows
BIN_SUFFIX=.exe
WINDRES=windres
WINDRES_TEMP:=$(WINDRES)
WINLIB=lib/$(PLATFORM_BIN)
WINDLL=../bin/$(PLATFORM_BIN)
override WINDRES=$(TOOLSET_PREFIX)$(WINDRES_TEMP)
ifneq (,$(findstring 64,$(PLATFORM)))
override WINDRES+= -F pe-x86-64
else
override WINDRES+= -F pe-i386
endif
STD_LIBS= -static-libgcc -static-libstdc++
CLIENT_INCLUDES= $(INCLUDES) -Iinclude
CLIENT_LIBS= -mwindows $(STD_LIBS) -L$(WINDLL) -L$(WINLIB) -lSDL2 -lSDL2_image -lSDL2_mixer -lzlib1 -lopengl32 -lenet -lws2_32 -lwinmm
else
APPMODIFIER=
ifneq (,$(findstring linux,$(PLATFORM)))
BIN_SUFFIX=_linux
else
ifneq (,$(findstring bsd,$(PLATFORM)))
BIN_SUFFIX=_bsd
ifneq (,$(findstring openbsd,$(PLATFORM)))
RMFLAGS= -f
endif
else
BIN_SUFFIX=_native
endif
endif
CLIENT_INCLUDES= $(INCLUDES) -I/usr/X11R6/include `sdl2-config --cflags`
CLIENT_LIBS= -Lenet -lenet -L/usr/X11R6/lib -lX11 `sdl2-config --libs` -lSDL2_image -lSDL2_mixer -lz -lGL
endif
ifneq (,$(findstring linux,$(PLATFORM)))
CLIENT_LIBS+= -lrt
else
ifneq (,$(findstring gnu,$(PLATFORM)))
CLIENT_LIBS+= -lrt
endif
endif
CLIENT_OBJS= \
	shared/crypto.o \
	shared/geom.o \
	shared/glemu.o \
	shared/stream.o \
	shared/tools.o \
	shared/zip.o \
	support/sqlite3.o \
	engine/bih.o \
	engine/blend.o \
	engine/blob.o \
	engine/client.o \
	engine/command.o \
	engine/console.o \
	engine/decal.o \
	engine/dynlight.o \
	engine/glare.o \
	engine/grass.o \
	engine/irc.o	\
	engine/lightmap.o \
	engine/main.o \
	engine/master.o \
	engine/material.o \
	engine/menus.o \
	engine/movie.o \
	engine/normal.o \
	engine/octa.o \
	engine/octaedit.o \
	engine/octarender.o \
	engine/physics.o \
	engine/pvs.o \
	engine/rendergl.o \
	engine/rendermodel.o \
	engine/renderparticles.o \
	engine/rendersky.o \
	engine/rendertext.o \
	engine/renderva.o \
	engine/server.o \
	engine/serverbrowser.o \
	engine/shader.o \
	engine/shadowmap.o \
	engine/sound.o \
	engine/texture.o \
	engine/ui.o \
	engine/water.o \
	engine/world.o \
	engine/worldio.o \
	game/ai.o \
	game/client.o \
	game/capture.o \
	game/defend.o \
	game/bomber.o \
	game/entities.o \
	game/game.o \
	game/hud.o \
	game/physics.o \
	game/projs.o \
	game/scoreboard.o \
	game/server.o \
	game/waypoint.o \
	game/weapons.o

CLIENT_PCH= shared/cube.h.gch engine/engine.h.gch

ifneq (,$(findstring mingw,$(PLATFORM)))
SERVER_INCLUDES= -DSTANDALONE $(INCLUDES) -Iinclude
SERVER_LIBS= -mwindows $(STD_LIBS) -L$(WINDLL) -L$(WINLIB) -lzlib1 -lenet -lws2_32 -lwinmm
else
SERVER_INCLUDES= -DSTANDALONE $(INCLUDES)
SERVER_LIBS= -Lenet -lenet -lz
endif
SERVER_OBJS= \
	shared/crypto-standalone.o \
	shared/geom-standalone.o \
	shared/stream-standalone.o \
	shared/tools-standalone.o \
	shared/zip-standalone.o \
	support/sqlite3.o \
	engine/command-standalone.o \
	engine/irc-standalone.o \
	engine/master-standalone.o \
	engine/server-standalone.o \
	game/server-standalone.o

LIBENET= enet/libenet.a

GENKEY_OBJS= engine/genkey.o shared/crypto-standalone.o

all:

default: all

clean-enet:
	$(MAKE) -C enet clean

clean-client:
	@rm $(RMFLAGS) $(CLIENT_PCH) $(CLIENT_OBJS) $(APPCLIENT)$(BIN_SUFFIX)

clean-server:
	@rm $(RMFLAGS) $(SERVER_OBJS) $(APPSERVER)$(BIN_SUFFIX)

clean-genkey:
	@rm $(RMFLAGS) $(GENKEY_OBJS) genkey$(BIN_SUFFIX)

clean: clean-enet clean-client clean-server clean-genkey

%.h.gch: %.h
	$(CXX) $(CXXFLAGS) -x c++-header -o $(subst .h.gch,.tmp.h.gch,$@) $(subst .h.gch,.h,$@)
	$(MV) $(subst .h.gch,.tmp.h.gch,$@) $@

%-standalone.o: %.cpp
	$(CXX) $(CXXFLAGS) -c -o $@ $(subst -standalone.o,.cpp,$@)

support/sqlite3.o: support/sqlite3.c
	$(CC) -DSQLITE_THREADSAFE=0 -DSQLITE_OMIT_LOAD_EXTENSION -c -o support/sqlite3.o support/sqlite3.c

$(CLIENT_OBJS): CXXFLAGS += $(CLIENT_INCLUDES)
$(filter shared/%,$(CLIENT_OBJS)): $(filter shared/%,$(CLIENT_PCH))
$(filter engine/%,$(CLIENT_OBJS)): $(filter engine/%,$(CLIENT_PCH))
$(filter game/%,$(CLIENT_OBJS)): $(filter game/%,$(CLIENT_PCH))

$(SERVER_OBJS): CXXFLAGS += $(SERVER_INCLUDES)

$(APPCLIENT)_windows$(BIN_SUFFIX): $(CLIENT_OBJS)
	$(WINDRES) -i $(APPNAME).rc -J rc -o $(APPNAME).res -O coff
	$(CXX) $(CXXFLAGS) -o $@ $(APPNAME).res $(CLIENT_OBJS) $(CLIENT_LIBS)

$(APPSERVER)_windows$(BIN_SUFFIX): $(SERVER_OBJS)
	$(WINDRES) -i $(APPNAME).rc -J rc -o $(APPNAME).res -O coff
	$(CXX) $(CXXFLAGS) -o $@ $(APPNAME).res $(SERVER_OBJS) $(SERVER_LIBS)

$(APPCLIENT)$(BIN_SUFFIX): $(LIBENET) $(CLIENT_OBJS)
	$(CXX) $(CXXFLAGS) $(LDFLAGS) -o $@ $(CLIENT_OBJS) $(CLIENT_LIBS)

$(APPSERVER)$(BIN_SUFFIX): $(LIBENET) $(SERVER_OBJS)
	$(CXX) $(CXXFLAGS) $(LDFLAGS) -o $@ $(SERVER_OBJS) $(SERVER_LIBS)

client: $(APPCLIENT)$(APPMODIFIER)$(BIN_SUFFIX)

server: $(APPSERVER)$(APPMODIFIER)$(BIN_SUFFIX)

genkey: genkey$(BIN_SUFFIX)

genkey$(BIN_SUFFIX): $(GENKEY_OBJS)
	$(CXX) $(CXXFLAGS) -o genkey$(BIN_SUFFIX) $(GENKEY_OBJS)

engine/genkey.o: engine/genkey.cpp
	$(CXX) $(CXXFLAGS) $(SERVER_INCLUDES) -c -o engine/genkey.o engine/genkey.cpp

$(INSTDIR)/%$(BIN_SUFFIX): %$(APPMODIFIER)$(BIN_SUFFIX)
	$(MKDIR) $(INSTDIR)
	$(CP) $< $@
ifneq (,$(STRIP))
	$(STRIP) $@
endif

$(INSTDIR)/genkey$(BIN_SUFFIX): genkey$(BIN_SUFFIX)
	$(MKDIR) $(INSTDIR)
	$(CP) $< $@
ifneq (,$(STRIP))
	$(STRIP) $@
endif

install-client: $(INSTDIR)/$(APPCLIENT)$(BIN_SUFFIX)

install-server: $(INSTDIR)/$(APPSERVER)$(BIN_SUFFIX)

install-genkey: $(INSTDIR)/genkey$(BIN_SUFFIX)

ifeq (,$(findstring mingw,$(PLATFORM)))
shared/cube2font.o: shared/cube2font.c
	$(CXX) $(CXXFLAGS) -c -o $@ $< `freetype-config --cflags`

cube2font: shared/cube2font.o
	$(CXX) $(CXXFLAGS) $(LDFLAGS) -o $@ $< `freetype-config --libs` -lz

install-cube2font: cube2font
	$(MKDIR) $(INSTDIR)
	install -m 755 $< $(INSTDIR)/$<
endif

install: install-client install-server install-genkey

$(LIBENET):
	$(MAKE) -C enet

depend:
	makedepend -Y -I. -Ishared -Iengine -Igame -Isupport $(subst .o,.cpp,$(CLIENT_OBJS))
	makedepend -a -o-standalone.o -Y -I. -Ishared -Iengine -Igame -Isupport -DSTANDALONE $(subst -standalone.o,.cpp,$(SERVER_OBJS))
	makedepend -a -o.h.gch -Y -I. -Ishared -Iengine -Igame -Isupport $(subst .h.gch,.h,$(CLIENT_PCH))

all: client server genkey

include system-install.mk
include dist.mk
include wiki.mk

engine/engine.h.gch: engine/version.h shared/cube.h.gch

# DO NOT DELETE

shared/crypto.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/crypto.o: shared/ents.h shared/glexts.h shared/glemu.h
shared/crypto.o: shared/iengine.h shared/igame.h
shared/geom.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/geom.o: shared/ents.h shared/glexts.h shared/glemu.h shared/iengine.h
shared/geom.o: shared/igame.h
shared/glemu.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/glemu.o: shared/ents.h shared/glexts.h shared/glemu.h shared/iengine.h
shared/glemu.o: shared/igame.h
shared/stream.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/stream.o: shared/ents.h shared/glexts.h shared/glemu.h
shared/stream.o: shared/iengine.h shared/igame.h
shared/tools.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/tools.o: shared/ents.h shared/glexts.h shared/glemu.h shared/iengine.h
shared/tools.o: shared/igame.h
shared/zip.o: shared/cube.h shared/tools.h shared/command.h shared/geom.h
shared/zip.o: shared/ents.h shared/glexts.h shared/glemu.h shared/iengine.h
shared/zip.o: shared/igame.h
engine/bih.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/bih.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/bih.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/bih.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/bih.o: engine/bih.h engine/texture.h engine/model.h
engine/blend.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/blend.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/blend.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/blend.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/blend.o: engine/bih.h engine/texture.h engine/model.h
engine/blob.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/blob.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/blob.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/blob.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/blob.o: engine/bih.h engine/texture.h engine/model.h
engine/client.o: engine/engine.h engine/version.h shared/cube.h
engine/client.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/client.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/client.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/client.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/client.o: engine/texture.h engine/model.h
engine/command.o: engine/engine.h engine/version.h shared/cube.h
engine/command.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/command.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/command.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/command.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/command.o: engine/texture.h engine/model.h
engine/console.o: engine/engine.h engine/version.h shared/cube.h
engine/console.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/console.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/console.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/console.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/console.o: engine/texture.h engine/model.h
engine/decal.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/decal.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/decal.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/decal.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/decal.o: engine/bih.h engine/texture.h engine/model.h
engine/dynlight.o: engine/engine.h engine/version.h shared/cube.h
engine/dynlight.o: shared/tools.h shared/command.h shared/geom.h
engine/dynlight.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/dynlight.o: shared/iengine.h shared/igame.h engine/irc.h
engine/dynlight.o: engine/sound.h engine/world.h engine/octa.h
engine/dynlight.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/dynlight.o: engine/model.h
engine/glare.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/glare.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/glare.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/glare.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/glare.o: engine/bih.h engine/texture.h engine/model.h
engine/glare.o: engine/rendertarget.h
engine/grass.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/grass.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/grass.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/grass.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/grass.o: engine/bih.h engine/texture.h engine/model.h
engine/irc.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/irc.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/irc.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/irc.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/irc.o: engine/bih.h engine/texture.h engine/model.h
engine/lightmap.o: engine/engine.h engine/version.h shared/cube.h
engine/lightmap.o: shared/tools.h shared/command.h shared/geom.h
engine/lightmap.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/lightmap.o: shared/iengine.h shared/igame.h engine/irc.h
engine/lightmap.o: engine/sound.h engine/world.h engine/octa.h
engine/lightmap.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/lightmap.o: engine/model.h
engine/main.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/main.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/main.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/main.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/main.o: engine/bih.h engine/texture.h engine/model.h
engine/master.o: engine/engine.h engine/version.h shared/cube.h
engine/master.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/master.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/master.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/master.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/master.o: engine/texture.h engine/model.h support/sqlite3.h
engine/material.o: engine/engine.h engine/version.h shared/cube.h
engine/material.o: shared/tools.h shared/command.h shared/geom.h
engine/material.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/material.o: shared/iengine.h shared/igame.h engine/irc.h
engine/material.o: engine/sound.h engine/world.h engine/octa.h
engine/material.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/material.o: engine/model.h
engine/menus.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/menus.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/menus.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/menus.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/menus.o: engine/bih.h engine/texture.h engine/model.h
engine/movie.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/movie.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/movie.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/movie.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/movie.o: engine/bih.h engine/texture.h engine/model.h
engine/normal.o: engine/engine.h engine/version.h shared/cube.h
engine/normal.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/normal.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/normal.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/normal.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/normal.o: engine/texture.h engine/model.h
engine/octa.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/octa.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/octa.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/octa.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/octa.o: engine/bih.h engine/texture.h engine/model.h
engine/octaedit.o: engine/engine.h engine/version.h shared/cube.h
engine/octaedit.o: shared/tools.h shared/command.h shared/geom.h
engine/octaedit.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/octaedit.o: shared/iengine.h shared/igame.h engine/irc.h
engine/octaedit.o: engine/sound.h engine/world.h engine/octa.h
engine/octaedit.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/octaedit.o: engine/model.h
engine/octarender.o: engine/engine.h engine/version.h shared/cube.h
engine/octarender.o: shared/tools.h shared/command.h shared/geom.h
engine/octarender.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/octarender.o: shared/iengine.h shared/igame.h engine/irc.h
engine/octarender.o: engine/sound.h engine/world.h engine/octa.h
engine/octarender.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/octarender.o: engine/model.h
engine/physics.o: engine/engine.h engine/version.h shared/cube.h
engine/physics.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/physics.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/physics.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/physics.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/physics.o: engine/texture.h engine/model.h engine/mpr.h
engine/pvs.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/pvs.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/pvs.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/pvs.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/pvs.o: engine/bih.h engine/texture.h engine/model.h
engine/rendergl.o: engine/engine.h engine/version.h shared/cube.h
engine/rendergl.o: shared/tools.h shared/command.h shared/geom.h
engine/rendergl.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/rendergl.o: shared/iengine.h shared/igame.h engine/irc.h
engine/rendergl.o: engine/sound.h engine/world.h engine/octa.h
engine/rendergl.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/rendergl.o: engine/model.h
engine/rendermodel.o: engine/engine.h engine/version.h shared/cube.h
engine/rendermodel.o: shared/tools.h shared/command.h shared/geom.h
engine/rendermodel.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/rendermodel.o: shared/iengine.h shared/igame.h engine/irc.h
engine/rendermodel.o: engine/sound.h engine/world.h engine/octa.h
engine/rendermodel.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/rendermodel.o: engine/model.h engine/ragdoll.h engine/animmodel.h
engine/rendermodel.o: engine/vertmodel.h engine/skelmodel.h engine/md2.h
engine/rendermodel.o: engine/md3.h engine/md5.h engine/obj.h engine/smd.h
engine/rendermodel.o: engine/iqm.h
engine/renderparticles.o: engine/engine.h engine/version.h shared/cube.h
engine/renderparticles.o: shared/tools.h shared/command.h shared/geom.h
engine/renderparticles.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/renderparticles.o: shared/iengine.h shared/igame.h engine/irc.h
engine/renderparticles.o: engine/sound.h engine/world.h engine/octa.h
engine/renderparticles.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/renderparticles.o: engine/model.h engine/rendertarget.h
engine/renderparticles.o: engine/depthfx.h engine/lensflare.h
engine/renderparticles.o: engine/explosion.h engine/lightning.h
engine/rendersky.o: engine/engine.h engine/version.h shared/cube.h
engine/rendersky.o: shared/tools.h shared/command.h shared/geom.h
engine/rendersky.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/rendersky.o: shared/iengine.h shared/igame.h engine/irc.h
engine/rendersky.o: engine/sound.h engine/world.h engine/octa.h
engine/rendersky.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/rendersky.o: engine/model.h
engine/rendertext.o: engine/engine.h engine/version.h shared/cube.h
engine/rendertext.o: shared/tools.h shared/command.h shared/geom.h
engine/rendertext.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/rendertext.o: shared/iengine.h shared/igame.h engine/irc.h
engine/rendertext.o: engine/sound.h engine/world.h engine/octa.h
engine/rendertext.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/rendertext.o: engine/model.h
engine/renderva.o: engine/engine.h engine/version.h shared/cube.h
engine/renderva.o: shared/tools.h shared/command.h shared/geom.h
engine/renderva.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/renderva.o: shared/iengine.h shared/igame.h engine/irc.h
engine/renderva.o: engine/sound.h engine/world.h engine/octa.h
engine/renderva.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/renderva.o: engine/model.h
engine/server.o: engine/engine.h engine/version.h shared/cube.h
engine/server.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/server.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/server.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/server.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/server.o: engine/texture.h engine/model.h
engine/serverbrowser.o: engine/engine.h engine/version.h shared/cube.h
engine/serverbrowser.o: shared/tools.h shared/command.h shared/geom.h
engine/serverbrowser.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/serverbrowser.o: shared/iengine.h shared/igame.h engine/irc.h
engine/serverbrowser.o: engine/sound.h engine/world.h engine/octa.h
engine/serverbrowser.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/serverbrowser.o: engine/model.h
engine/shader.o: engine/engine.h engine/version.h shared/cube.h
engine/shader.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/shader.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/shader.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/shader.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/shader.o: engine/texture.h engine/model.h
engine/shadowmap.o: engine/engine.h engine/version.h shared/cube.h
engine/shadowmap.o: shared/tools.h shared/command.h shared/geom.h
engine/shadowmap.o: shared/ents.h shared/glexts.h shared/glemu.h
engine/shadowmap.o: shared/iengine.h shared/igame.h engine/irc.h
engine/shadowmap.o: engine/sound.h engine/world.h engine/octa.h
engine/shadowmap.o: engine/lightmap.h engine/bih.h engine/texture.h
engine/shadowmap.o: engine/model.h engine/rendertarget.h
engine/sound.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/sound.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/sound.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/sound.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/sound.o: engine/bih.h engine/texture.h engine/model.h
engine/texture.o: engine/engine.h engine/version.h shared/cube.h
engine/texture.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/texture.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/texture.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/texture.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/texture.o: engine/texture.h engine/model.h engine/scale.h
engine/ui.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/ui.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/ui.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/ui.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/ui.o: engine/bih.h engine/texture.h engine/model.h engine/textedit.h
engine/water.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/water.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/water.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/water.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/water.o: engine/bih.h engine/texture.h engine/model.h
engine/world.o: engine/engine.h engine/version.h shared/cube.h shared/tools.h
engine/world.o: shared/command.h shared/geom.h shared/ents.h shared/glexts.h
engine/world.o: shared/glemu.h shared/iengine.h shared/igame.h engine/irc.h
engine/world.o: engine/sound.h engine/world.h engine/octa.h engine/lightmap.h
engine/world.o: engine/bih.h engine/texture.h engine/model.h
engine/worldio.o: engine/engine.h engine/version.h shared/cube.h
engine/worldio.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
engine/worldio.o: shared/glexts.h shared/glemu.h shared/iengine.h
engine/worldio.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
engine/worldio.o: engine/octa.h engine/lightmap.h engine/bih.h
engine/worldio.o: engine/texture.h engine/model.h
game/ai.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/ai.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/ai.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/ai.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/ai.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/ai.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/ai.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/ai.o: game/capture.h game/defend.h game/bomber.h
game/client.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/client.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/client.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/client.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/client.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/client.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/client.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/client.o: game/capture.h game/defend.h game/bomber.h
game/capture.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/capture.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/capture.o: shared/glexts.h shared/glemu.h shared/iengine.h
game/capture.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
game/capture.o: engine/octa.h engine/lightmap.h engine/bih.h engine/texture.h
game/capture.o: engine/model.h game/gamemode.h game/weapons.h game/weapdef.h
game/capture.o: game/player.h game/teamdef.h game/playerdef.h game/vars.h
game/capture.o: game/ai.h game/capture.h game/defend.h game/bomber.h
game/defend.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/defend.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/defend.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/defend.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/defend.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/defend.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/defend.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/defend.o: game/capture.h game/defend.h game/bomber.h
game/bomber.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/bomber.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/bomber.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/bomber.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/bomber.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/bomber.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/bomber.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/bomber.o: game/capture.h game/defend.h game/bomber.h
game/entities.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/entities.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/entities.o: shared/glexts.h shared/glemu.h shared/iengine.h
game/entities.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
game/entities.o: engine/octa.h engine/lightmap.h engine/bih.h
game/entities.o: engine/texture.h engine/model.h game/gamemode.h
game/entities.o: game/weapons.h game/weapdef.h game/player.h game/teamdef.h
game/entities.o: game/playerdef.h game/vars.h game/ai.h game/capture.h
game/entities.o: game/defend.h game/bomber.h
game/game.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/game.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/game.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/game.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/game.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/game.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/game.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/game.o: game/capture.h game/defend.h game/bomber.h
game/hud.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/hud.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/hud.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/hud.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/hud.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/hud.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/hud.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/hud.o: game/capture.h game/defend.h game/bomber.h game/compass.h
game/physics.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/physics.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/physics.o: shared/glexts.h shared/glemu.h shared/iengine.h
game/physics.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
game/physics.o: engine/octa.h engine/lightmap.h engine/bih.h engine/texture.h
game/physics.o: engine/model.h game/gamemode.h game/weapons.h game/weapdef.h
game/physics.o: game/player.h game/teamdef.h game/playerdef.h game/vars.h
game/physics.o: game/ai.h game/capture.h game/defend.h game/bomber.h
game/projs.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/projs.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/projs.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/projs.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/projs.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/projs.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/projs.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/projs.o: game/capture.h game/defend.h game/bomber.h
game/scoreboard.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/scoreboard.o: shared/tools.h shared/command.h shared/geom.h
game/scoreboard.o: shared/ents.h shared/glexts.h shared/glemu.h
game/scoreboard.o: shared/iengine.h shared/igame.h engine/irc.h
game/scoreboard.o: engine/sound.h engine/world.h engine/octa.h
game/scoreboard.o: engine/lightmap.h engine/bih.h engine/texture.h
game/scoreboard.o: engine/model.h game/gamemode.h game/weapons.h
game/scoreboard.o: game/weapdef.h game/player.h game/teamdef.h
game/scoreboard.o: game/playerdef.h game/vars.h game/ai.h game/capture.h
game/scoreboard.o: game/defend.h game/bomber.h
game/server.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/server.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/server.o: shared/glexts.h shared/glemu.h shared/iengine.h shared/igame.h
game/server.o: engine/irc.h engine/sound.h engine/world.h engine/octa.h
game/server.o: engine/lightmap.h engine/bih.h engine/texture.h engine/model.h
game/server.o: game/gamemode.h game/weapons.h game/weapdef.h game/player.h
game/server.o: game/teamdef.h game/playerdef.h game/vars.h game/ai.h
game/server.o: game/capture.h game/defend.h game/bomber.h game/auth.h
game/server.o: game/capturemode.h game/defendmode.h game/bombermode.h
game/server.o: game/duelmut.h game/aiman.h
game/waypoint.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/waypoint.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/waypoint.o: shared/glexts.h shared/glemu.h shared/iengine.h
game/waypoint.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
game/waypoint.o: engine/octa.h engine/lightmap.h engine/bih.h
game/waypoint.o: engine/texture.h engine/model.h game/gamemode.h
game/waypoint.o: game/weapons.h game/weapdef.h game/player.h game/teamdef.h
game/waypoint.o: game/playerdef.h game/vars.h game/ai.h game/capture.h
game/waypoint.o: game/defend.h game/bomber.h
game/weapons.o: game/game.h engine/engine.h engine/version.h shared/cube.h
game/weapons.o: shared/tools.h shared/command.h shared/geom.h shared/ents.h
game/weapons.o: shared/glexts.h shared/glemu.h shared/iengine.h
game/weapons.o: shared/igame.h engine/irc.h engine/sound.h engine/world.h
game/weapons.o: engine/octa.h engine/lightmap.h engine/bih.h engine/texture.h
game/weapons.o: engine/model.h game/gamemode.h game/weapons.h game/weapdef.h
game/weapons.o: game/player.h game/teamdef.h game/playerdef.h game/vars.h
game/weapons.o: game/ai.h game/capture.h game/defend.h game/bomber.h

shared/crypto-standalone.o: shared/cube.h shared/tools.h shared/command.h
shared/crypto-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
shared/crypto-standalone.o: shared/igame.h
shared/geom-standalone.o: shared/cube.h shared/tools.h shared/command.h
shared/geom-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
shared/geom-standalone.o: shared/igame.h
shared/stream-standalone.o: shared/cube.h shared/tools.h shared/command.h
shared/stream-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
shared/stream-standalone.o: shared/igame.h
shared/tools-standalone.o: shared/cube.h shared/tools.h shared/command.h
shared/tools-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
shared/tools-standalone.o: shared/igame.h
shared/zip-standalone.o: shared/cube.h shared/tools.h shared/command.h
shared/zip-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
shared/zip-standalone.o: shared/igame.h
engine/command-standalone.o: engine/engine.h engine/version.h shared/cube.h
engine/command-standalone.o: shared/tools.h shared/command.h shared/geom.h
engine/command-standalone.o: shared/ents.h shared/iengine.h shared/igame.h
engine/command-standalone.o: engine/irc.h engine/sound.h
engine/irc-standalone.o: engine/engine.h engine/version.h shared/cube.h
engine/irc-standalone.o: shared/tools.h shared/command.h shared/geom.h
engine/irc-standalone.o: shared/ents.h shared/iengine.h shared/igame.h
engine/irc-standalone.o: engine/irc.h engine/sound.h
engine/master-standalone.o: engine/engine.h engine/version.h shared/cube.h
engine/master-standalone.o: shared/tools.h shared/command.h shared/geom.h
engine/master-standalone.o: shared/ents.h shared/iengine.h shared/igame.h
engine/master-standalone.o: engine/irc.h engine/sound.h support/sqlite3.h
engine/server-standalone.o: engine/engine.h engine/version.h shared/cube.h
engine/server-standalone.o: shared/tools.h shared/command.h shared/geom.h
engine/server-standalone.o: shared/ents.h shared/iengine.h shared/igame.h
engine/server-standalone.o: engine/irc.h engine/sound.h
game/server-standalone.o: game/game.h engine/engine.h engine/version.h
game/server-standalone.o: shared/cube.h shared/tools.h shared/command.h
game/server-standalone.o: shared/geom.h shared/ents.h shared/iengine.h
game/server-standalone.o: shared/igame.h engine/irc.h engine/sound.h
game/server-standalone.o: game/gamemode.h game/weapons.h game/weapdef.h
game/server-standalone.o: game/player.h game/teamdef.h game/playerdef.h
game/server-standalone.o: game/vars.h game/capture.h game/defend.h
game/server-standalone.o: game/bomber.h game/auth.h game/capturemode.h
game/server-standalone.o: game/defendmode.h game/bombermode.h game/duelmut.h
game/server-standalone.o: game/aiman.h

shared/cube.h.gch: shared/tools.h shared/command.h shared/geom.h
shared/cube.h.gch: shared/ents.h shared/glexts.h shared/glemu.h
shared/cube.h.gch: shared/iengine.h shared/igame.h
engine/engine.h.gch: engine/version.h shared/cube.h shared/tools.h
engine/engine.h.gch: shared/command.h shared/geom.h shared/ents.h
engine/engine.h.gch: shared/glexts.h shared/glemu.h shared/iengine.h
engine/engine.h.gch: shared/igame.h engine/irc.h engine/sound.h
engine/engine.h.gch: engine/world.h engine/octa.h engine/lightmap.h
engine/engine.h.gch: engine/bih.h engine/texture.h engine/model.h
