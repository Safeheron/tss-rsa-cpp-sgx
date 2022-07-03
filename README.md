# sgx-tss-rsa-cpp

git clone --recurse-submodules https://github.com/safeheron/tss-rsa-cpp.git

cd tss-rsa-cpp
mkdir sgx
touch sgx.sh
patch  -d/ -p1 < ../tss.patch

cd sgx
chmod +x sgx.sh
./sgx.sh
