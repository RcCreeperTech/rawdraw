all : index.html

#For tools (Works in Ubuntu 20.04 (Including WSL), Mint)
# sudo apt-get install clang-10 lld-10 binaryen
# sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-10 20
# sudo update-alternatives --install /usr/bin/wasm-ld wasm-ld /usr/bin/wasm-ld-10 20

#node-uglify lld clang-10 lld-10 binaryen

#Path to rawdraw
CFLAGS:=-I..

CLANG?=clang
WASMOPT?=wasm-opt
TERSER?=terser
TERSERFLAGS?= -ecma 2017 -d RAWDRAW_USE_LOOP_FUNCTION=false -d RAWDRAW_NEED_BLITTER=true

CFLAGS+=-DWASM -nostdlib --target=wasm32 \
		-flto -Oz \
		-Wl,--lto-O3 \
		-Wl,--no-entry \
		-Wl,--allow-undefined \
		-Wl,--import-memory

WOFLAGS:=--asyncify --pass-arg=asyncify-imports@bynsyncify.* --pass-arg=asyncify-ignore-indirect

opt.js : template.js main.wasm
	cat main.wasm | base64 -w 0 > main.b64
	sed 's/$${BLOB}/\n$${BLOB}\n/g' template.js | sed $$'/$${BLOB}/{r main.b64\nd}' | sed -z "s/'\n/'/" > mid.js
	rm main.b64
	#Comment the below line out if you don't want to uglify the javascript.
	$(TERSER) $(TERSERFLAGS) mid.js -o opt.js
	rm mid.js

index.html : template.ht opt.js
	sed $$'/$${JAVASCRIPT_DATA}/{r opt.js\nd}' template.ht > $@

main.wasm: rawdraw.c
	$(CLANG) $(CFLAGS) $^ -o $@
	$(WASMOPT) $(WOFLAGS) -Oz main.wasm -o main.wasm
	#wasm-objdump -d main.wasm > main.disassembly.txt

.PHONY: clean
clean:
	rm -rf main.wasm opt.js index.html
