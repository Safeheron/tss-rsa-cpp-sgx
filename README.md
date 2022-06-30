# sgx-tss-rsa-cpp
# Pass --recurse-submodules to the git clone command, and it will automatically initialize and update each submodule in the repository, including nested submodules if any of the submodules in the repository have submodules themselves.
git clone --recurse-submodules https://github.com/safeheron/tss-rsa-cpp.git
cd tss-rsa-cpp
patch  -d/ -p1 < ../tss.patch
cd sgx
./sgx
