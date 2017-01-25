# Static buildpack

Baed on Nginx 1.11.8, LuaJIT 2.0.4, OpenSSL 1.0.2j

## Build instructions
```shell
wget http://nginx.org/download/nginx-1.11.8.tar.gz
tar xvfz nginx-1.11.8.tar.gz
https://github.com/simpl/ngx_devel_kit/releases/tag/v0.3.0
tar xvfz v0.3.0.tgz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.7.tar.gz
tar xvfz v0.10.7.tar.gz
wget http://luajit.org/download/LuaJIT-2.0.4.tar.gz
tar  xvfz LuaJIT-2.0.4.tar.gz
wget https://www.openssl.org/source/openssl-1.0.2j.tar.gz
tar xvfz openssl-1.1.0b.tar.gz

docker run -i -v ${PWD}:/working_dir -w /working_dir -it cloudfoundry/cflinuxfs2 bash

cd LuaJIT-2.0.4
make
make install

cd ..

cd nginx-1.11.8

export LUAJIT_LIB=/usr/local/lib
export LUAJIT_INC=/usr/local/include/luajit-2.0

./configure --prefix=/ --error-log-path=stderr \
--with-http_v2_module \
--with-http_ssl_module \
--with-http_realip_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_auth_request_module \
--with-http_random_index_module \
--with-http_secure_link_module \
--with-http_stub_status_module \
--without-http_uwsgi_module \
--without-http_scgi_module \
--with-pcre \
--with-pcre-jit \
--with-http_sub_module \
--with-ld-opt="-Wl,-rpath,/usr/local/lib" \
--add-module=../lua-nginx-module-0.10.7 \
--add-module=../ngx_devel_kit-0.3.0 --with-openssl=../openssl-1.0.2j
```
Update objs/Makefile (instead of dynamic lua (-llua), put just /usr/local/lib/libluajit-5.1.a to link it statically)
```
  -L/usr/local/lib -Wl,-rpath,/usr/local/lib -Wl,-E -ldl -lpthread -lcrypt -L/usr/local/lib \
  /usr/local/lib/libluajit-5.1.a -lm -ldl -lpcre \
  ../openssl-1.0.2j/.openssl/lib/libssl.a ../openssl-1.0.2j/.openssl/lib/libcrypto.a -ldl -lz \
  -Wl,-E
```
Now build it:
```
make -j2
cd objs
strip nginx
```
Reference:

https://github.com/openresty/lua-nginx-module#installation







