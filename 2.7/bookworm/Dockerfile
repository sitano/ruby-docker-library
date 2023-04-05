#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM buildpack-deps:bookworm

# skip installing gem documentation
RUN set -eux; \
	mkdir -p /usr/local/etc; \
	{ \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /usr/local/etc/gemrc

ENV LANG C.UTF-8
ENV RUBY_MAJOR 2.7
ENV RUBY_VERSION 2.7.8
ENV RUBY_DOWNLOAD_SHA256 f22f662da504d49ce2080e446e4bea7008cee11d5ec4858fc69000d0e5b1d7fb

# some of ruby's build scripts are written in ruby
#   we purge system ruby later to make sure our final image uses what we just built
RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		bison \
		dpkg-dev \
		libgdbm-dev \
		ruby \
	; \
	rm -rf /var/lib/apt/lists/*; \
	\
	wget -O /tmp/libssl1.1_1.1.1n-0+deb11u4_amd64.deb "http://ftp.debian.org/debian/pool/main/o/openssl/libssl1.1_1.1.1n-0+deb11u4_amd64.deb"; \
	wget -O /tmp/libssl-dev_1.1.1n-0+deb11u4_amd64.deb "http://ftp.debian.org/debian/pool/main/o/openssl/libssl-dev_1.1.1n-0+deb11u4_amd64.deb"; \
	\
	dpkg -i /tmp/libssl1.1_1.1.1n-0+deb11u4_amd64.deb; \
	dpkg -i /tmp/libssl-dev_1.1.1n-0+deb11u4_amd64.deb; \
	\
	echo "62faed6afef01c09524a5f082bd63f1732c5f986856be11a3e7401128c5c01c9 /tmp/libssl1.1_1.1.1n-0+deb11u4_amd64.deb" | sha256sum --check --strict; \
	echo "58b94df14743d35ac280d35cdc29d90548e55ab6e9b7399d070ab7f2e4f6f11f /tmp/libssl-dev_1.1.1n-0+deb11u4_amd64.deb" | sha256sum --check --strict; \
	\
	rm /tmp/libssl1.1_1.1.1n-0+deb11u4_amd64.deb; \
	rm /tmp/libssl-dev_1.1.1n-0+deb11u4_amd64.deb; \
	\
	wget -O ruby.tar.xz "https://cache.ruby-lang.org/pub/ruby/${RUBY_MAJOR%-rc}/ruby-$RUBY_VERSION.tar.xz"; \
	echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.xz" | sha256sum --check --strict; \
	\
	mkdir -p /usr/src/ruby; \
	tar -xJf ruby.tar.xz -C /usr/src/ruby --strip-components=1; \
	rm ruby.tar.xz; \
	\
	cd /usr/src/ruby; \
	\
# hack in "ENABLE_PATH_CHECK" disabling to suppress:
#   warning: Insecure world writable dir
	{ \
		echo '#define ENABLE_PATH_CHECK 0'; \
		echo; \
		cat file.c; \
	} > file.c.new; \
	mv file.c.new file.c; \
	\
	autoconf; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	./configure \
		--build="$gnuArch" \
		--disable-install-doc \
		--enable-shared \
	; \
	make -j "$(nproc)"; \
	make install; \
	\
# fix CVE-2020-36327 and CVE-2021-43809 of 2.1.4
	rm /usr/local/lib/ruby/gems/2.7.0/specifications/default/bundler-2.1.4.gemspec; \
	rm -rf /usr/local/lib/ruby/gems/2.7.0/gems/bundler-2.1.4; \
	gem install bundler -v 2.2.33 --default; \
	gem install bundler -v 2.2.33; \
# fix CVE-2021-31799 of 6.2.1.1
	rm /usr/local/lib/ruby/gems/2.7.0/specifications/default/rdoc-6.2.1.1.gemspec; \
	rm -rf /usr/local/lib/ruby/gems/2.7.0/gems/rdoc-6.2.1.1; \
	gem install rdoc -v 6.3.1 --default; \
	gem install rdoc -v 6.3.1; \
# fix CVE-2021-28965 of 3.2.3.1
	rm /usr/local/lib/ruby/gems/2.7.0/specifications/default/rexml-3.2.3.1.gemspec; \
	rm -rf /usr/local/lib/ruby/gems/2.7.0/gems/rexml-3.2.3.1; \
	gem install rexml -v 3.2.5 --default; \
	gem install rexml -v 3.2.5; \
	\
	apt-mark auto '.*' > /dev/null; \
	apt-mark manual $savedAptMark > /dev/null; \
	find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
		| awk '/=>/ { print $(NF-1) }' \
		| sort -u \
		| grep -vE '^/usr/local/lib/' \
		| xargs -r dpkg-query --search \
		| cut -d: -f1 \
		| sort -u \
		| xargs -r apt-mark manual \
	; \
	apt-mark manual libssl1.1; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
	cd /; \
	rm -r /usr/src/ruby; \
# verify we have no "ruby" packages installed
	if dpkg -l | grep -i ruby; then exit 1; fi; \
	[ "$(command -v ruby)" = '/usr/local/bin/ruby' ]; \
# rough smoke test
	ruby --version; \
	gem --version; \
	bundle --version

# don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_SILENCE_ROOT_WARNING=1 \
	BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $GEM_HOME/bin:$PATH
# adjust permissions of a few directories for running "gem install" as an arbitrary user
RUN mkdir -p "$GEM_HOME" && chmod 1777 "$GEM_HOME"

CMD [ "irb" ]
