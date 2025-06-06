FROM demisto/py3-tools-ubi:1.0.0.114275




COPY requirements.txt .
COPY chromium/download_chromedriver.sh .
COPY repos /repos

ARG LEPTONICA_VERSION=1.82.0
ARG TESSERACT_VERSION=5.2.0
ARG ROCKY_BASE=rocky-base
ARG ROCKY_RELEASE=rocky-release
ARG ROCKY_DEVEL=rocky-devel
ARG FEDORA_EPEL=fedora-epel

# handles chromium + tesseract
# for tesseract - followed instructions from here: https://computingforgeeks.com/install-tesseract-ocr-on-rocky-almalinux/
# note: linux_signing_key.pub is using sha1 for verification which is not supported in ubi 9+, hence need to update crypto policies, more info at https://www.redhat.com/en/blog/rhel-security-sha-1-package-signatures-distrusted-rhel-9
# there is not linux_signing_key.pub from google that is not based on sha1
RUN dnf update --nodocs -y  \
    && dnf upgrade -y --nodocs \
    && dnf install --nodocs -y wget git \
    && wget https://dl-ssl.google.com/linux/linux_signing_key.pub \
    && update-crypto-policies --set DEFAULT:SHA1 \
    && rpm --import linux_signing_key.pub \
    && update-crypto-policies --set DEFAULT \
    && wget https://www.centos.org/keys/RPM-GPG-KEY-CentOS-Official \
    && rpm --import RPM-GPG-KEY-CentOS-Official \
    && mv repos/* /etc/yum.repos.d/ \
    && dnf install -y --nodocs python3-devel gcc gcc-c++ make wget git unzip llvm-libs libXpm iproute jq \
    && dnf install -y --nodocs --enablerepo=$ROCKY_BASE --enablerepo=$ROCKY_RELEASE tigervnc-server-minimal xorg-x11-utils google-chrome-stable iputils nmap bind-utils \
    && dnf install -y --nodocs --enablerepo=$ROCKY_BASE --enablerepo=$ROCKY_RELEASE --enablerepo=$FEDORA_EPEL ImageMagick \
    && ./download_chromedriver.sh \
    && touch /var/chrome_headless.log \
    && chmod 777 /var/chrome_headless.log \
    && touch /var/chrome_instances.json \
    && chmod 777 /var/chrome_instances.json \
    && unzip chromedriver-linux64.zip \
    && mv chromedriver-linux64/chromedriver /usr/bin/chromedriver \
    && rm -rf chromedriver-linux64.zip \
    && google-chrome --version \
    && chromedriver --version \
    && dnf install -y --enablerepo=$ROCKY_DEVEL --enablerepo=$ROCKY_BASE --nodocs git automake make autoconf libtool zlib clang zlib-devel libjpeg libjpeg-devel libwebp libwebp-devel \
    libtiff libtiff-devel libpng libpng-devel pango giflib giflib-devel dejavu-sans-mono-fonts \
    krb5-pkinit krb5-server krb5-server-ldap krb5-libs \
    && git clone --depth 1 -b $LEPTONICA_VERSION https://github.com/DanBloomberg/leptonica \
    && cd leptonica \
    && ./autogen.sh && ./configure && make && make install && ldconfig \
    && cd .. \
    && git clone --depth 1 -b $TESSERACT_VERSION https://github.com/tesseract-ocr/tesseract.git \
    && cd tesseract \
    && PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./autogen.sh  \
    && PKG_CONFIG_PATH=/usr/local/lib/pkgconfig ./configure  \
    && PKG_CONFIG_PATH=/usr/local/lib/pkgconfig gmake \
    && PKG_CONFIG_PATH=/usr/local/lib/pkgconfig make install \
    && ldconfig \
    && cd .. \
    && git clone https://github.com/zdenop/tessdata_downloader.git \
    && for lang in eng ara chi_sim chi_sim_vert chi_tra chi_tra_vert deu fra heb ita jpn jpn_vert pol por rus spa swe tur ; \
    do python tessdata_downloader/tessdata_downloader.py -o /usr/local/share/tessdata/ -l $lang; done \
    && tesseract --version --list-langs \
    && rm -rf tessdata_downloader leptonica tesseract download_chromedriver.sh repos \
    && dnf remove -y python3-devel gcc gcc-c++ make platform-python-pip jq unzip git make autoconf automake libtool libpng-devel libjpeg-devel libtiff-devel zlib-devel giflib-devel webkit2gtk3-jsc bind-libs.x86_64 polkit polkit-libs \
    && dnf clean all \
    && rm -rf /var/cache/dnf


# handles os dependencies for: crypto + readpdf + parse-emails + docxpy + sklearn + pandas + ippysocks-py3 + oauthlib
# unzip + py3ews + taxii2 + pan-os-python + slackv3 + google-api-py3 + boto3py3
# pyjwt3 + joe-security + slack + office-utils + auth-utils

# line 6 - since in ubi its 7za --> need to make it to run as 7z like our script
# lines 8-10 is the configuration for sklearn
# lines 12-14 - ssl fixes
RUN dnf install -y --nodocs --enablerepo=$ROCKY_RELEASE --nobest python3-devel gcc gcc-c++ make wget git poppler poppler-utils \
        rust cargo libxml2-devel libxslt-devel krb5-devel \
        libffi-devel openssl-devel libffi libSM mesa-libGL \
        openssh ca-certificates openssl less rsync libpng-devel freetype-devel gcc-gfortran openblas unzip libstdc++ \
        && dnf install -y --nodocs --enablerepo=$ROCKY_RELEASE libreoffice \
        && dnf remove -y postgresql-jdbc --noautoremove \
        && dnf install -y --nodocs --enablerepo=$FEDORA_EPEL p7zip unrar \
        && ln -s /usr/bin/7za /usr/bin/7z \
        && pip install --no-cache-dir -r requirements.txt \
        && mkdir /etc/ml \
        && python -c "import nltk; nltk.download('wordnet', download_dir='/etc/ml/nltk_data'); nltk.download('stopwords', download_dir='/etc/ml/nltk_data'); nltk.download('punkt', download_dir='/etc/ml/nltk_data')" \
        && chown -R demisto:demisto /etc/ml && chmod -R 775 /etc/ml \
        && dnf install -y --nodocs java-11-openjdk-headless cairo libX11-xcb \
        && sed -i 's/##legacy = legacy_sect/legacy = legacy_sect/' /etc/pki/tls/openssl.cnf \
        && sed -i 's/##\[legacy_sect\]/\[legacy_sect\]/' /etc/pki/tls/openssl.cnf \
        && sed -i 's/##activate = 1/activate = 1/' /etc/pki/tls/openssl.cnf \
        && dnf remove -y python3-devel rust cargo libffi-devel openssl-devel gcc gcc-c++ make git platform-python-pip krb5-devel \
        && dnf clean all \
        && rm -rf /var/cache/dnf /root/.cargo \
        && pip uninstall -y pcodedmp


ENV NLTK_DATA='/etc/ml/nltk_data'
