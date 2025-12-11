#RL10.1

# Create /etc/sudoers.d/clamscan 
midgard ALL=(root) NOPASSWD: /usr/bin/clamscan
midgard ALL=(root) NOPASSWD: /usr/bin/clamdscan

# Create /etc/sudoers.d/freshclam 
midgard ALL=(root) NOPASSWD: /usr/bin/freshclam

# Create /etc/sudoers.d/mysql 
midgard ALL=(root) NOPASSWD: /bin/mysql

# Create /etc/sudoers.d/maldet 
midgard ALL=(root) NOPASSWD: /usr/local/sbin/maldet

# Set Perms
chmod 0440 /etc/sudoers.d/*

# If this pass 
visudo -c

# Refresh sudo
visudo -c -f /etc/sudoers.d/maldet 
visudo -c -f /etc/sudoers.d/freshclam 
visudo -c -f /etc/sudoers.d/clamscan 
visudo -c -f /etc/sudoers.d/mysql 

dnf install mariadb-server
dnf install rsync
dnf install clamav clamav-update clamav-scanner-systemd
dnf install tar

# For Reporting
dnf install google-noto-emoji-fonts google-noto-sans-symbols2-fonts

# Lets clamdscan scan system
setsebool -P antivirus_can_scan_system 1

# :> /etc/clamd.d/scan.conf and add this content
LocalSocket /var/run/clamd.scan/clamd.sock
LocalSocketMode 666
LogFile /var/log/clamd.scan
LogTime yes
LogVerbose yes

# Signature DB directory
DatabaseDirectory /var/lib/clamav

# Restart clamav
systemctl restart clamd@scan.service
systemctl status clamd@scan.service


#### NOT SO SURE ABOUT PCRE & VECTORSCAN

# Build PCRE
dnf install -y bzip2-devel zlib-devel
cd /usr/local/src
sudo curl -LO https://ftp.pcre.org/pub/pcre/pcre-8.45.tar.gz
sudo tar xzf pcre-8.45.tar.gz
cd pcre-8.45
sudo ./configure --enable-jit --prefix=/usr/local
sudo make -j"$(nproc)"
sudo make install
echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/local.conf
sudo ldconfig
ldconfig -p | grep pcre

# VectorScan & Ragel Build First
  cd /usr/local/src
  # Download Ragel 6.10 from the URL you gave
  sudo curl -LO https://www.colm.net/files/ragel/ragel-6.10.tar.gz
  # Extract
  sudo tar xzf ragel-6.10.tar.gz
  cd ragel-6.10
  # Configure, build, install under /usr/local
  sudo ./configure --prefix=/usr/local
  sudo make -j"$(nproc)"
  sudo make install
# VectorScan
sudo dnf groupinstall "Development Tools" -y
sudo dnf install -y \
  gcc-c++ \
  cmake \
  ninja-build \
  automake \
  autoconf \
  libtool \
  python3-devel \
  boost-devel \
  git
cd /usr/local/src
sudo git clone https://github.com/VectorCamp/vectorscan.git
cd vectorscan
sudo rm -rf build
sudo mkdir build
cd build
sudo cmake .. -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=ON \
  -DCMAKE_INSTALL_PREFIX=/usr/local
sudo ninja
sudo ninja install
echo "/usr/local/lib" | sudo tee /etc/ld.so.conf.d/local.conf
sudo ldconfig
ldconfig -p | grep -i vector
# soemthing like that



# Install Wordfence-CLI
dnf install python3
dnf install python3-pip
pip3 install wordfence
/usr/local/bin/wordfence 
# Configure wordfence before usage

# Install Weasyprint
pip3 install weasyprint

# Install Maldet
curl -LO https://www.rfxn.com/downloads/maldetect-current.tar.gz
tar xvfz maldetect-current.tar.gz
cd maldetect-#.#.#
./install.sh
