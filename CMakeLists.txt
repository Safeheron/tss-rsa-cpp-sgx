cmake_minimum_required(VERSION 3.20)

project("CryptoTSSRSASgx" VERSION 1.0.1)

add_definitions(-DPB_ENABLE_SGX)

find_package(SafeheronCryptoSuitesSgx REQUIRED)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE "Release")

# Set all flags
set(SGX_COMMON_C_FLAGS "-m64 -O2")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -nostdinc -fmacro-prefix-map=${CMAKE_SOURCE_DIR}=/safeheron")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_C_FLAGS} -nostdinc++ -fmacro-prefix-map=${CMAKE_SOURCE_DIR}=/safeheron")

# Set the sgxsdk path
set(SGX_SDK_PATH "/opt/intel/sgxsdk")
set(SGX_SDK_INCLUDE_DIR "${SGX_SDK_PATH}/include")
set(SGX_SDK_LIBRARIES_DIR "${SGX_SDK_PATH}/lib64")
set(SGX_C_INCLUDE_DIR "${SGX_SDK_INCLUDE_DIR}/tlibc")
set(SGX_CXX_INCLUDE_DIR "${SGX_SDK_INCLUDE_DIR}/libcxx")
set(SGX_PROTOBUF_INCLUDE_DIR "${SGX_SDK_INCLUDE_DIR}/tprotobuf")

# Set the sgxssl path
set(SGX_SSL_PATH "/opt/intel/sgxssl")
set(SGX_SSL_INCLUDE_DIR "${SGX_SSL_PATH}/include")
set(SGX_SSL_LIBRARIES_DIR "${SGX_SSL_PATH}/lib64")
set(TRUST_SGX_SSL_LIBRARY "${SGX_SSL_LIBRARIES_DIR}/libsgx_tsgxssl.a")
set(TRUST_SGX_SSL_CRYPTO_LIBRARY "${SGX_SSL_LIBRARIES_DIR}/libsgx_tsgxssl_crypto.a")
set(UNTRUST_SGX_SSL_LIBRARY "${SGX_SSL_LIBRARIES_DIR}/libsgx_usgxssl.a")

# Set source file path
set(TSS_RSA_SOURCE_PATH "src/tss-rsa-cpp/src/crypto-tss-rsa")

AUX_SOURCE_DIRECTORY(${TSS_RSA_SOURCE_PATH} TSS_RSA_SOURCE)

add_library(${CMAKE_PROJECT_NAME} STATIC
        ${TSS_RSA_SOURCE}
        ${TSS_RSA_SOURCE_PATH}/proto_gen/tss_rsa.pb.switch.cc)

# Additional include paths
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
        ${SafeheronCryptoSuitesSgx_INCLUDE_DIRS}
        ${SGX_SSL_INCLUDE_DIR}
        ${SGX_SDK_INCLUDE_DIR}
        ${SGX_C_INCLUDE_DIR}
        ${SGX_CXX_INCLUDE_DIR}
        ${SGX_PROTOBUF_INCLUDE_DIR}
        )

# Link flags
target_link_libraries(${CMAKE_PROJECT_NAME} "${SGX_COMMON_C_FLAGS} \
        -Wl,--no-undefined -nostdlib -nodefaultlibs -nostartfiles -L${SGX_SDK_LIBRARIES_DIR} \
        -Wl,--whole-archive –lsgx_trts ${TRUST_SGX_SSL_LIBRARY} -Wl,--no-whole-archive \
        -Wl,--start-group ${TRUST_SGX_SSL_CRYPTO_LIBRARY} ${SafeheronCryptoSuitesSgx_LIBRARY} -lsgx_pthread -lsgx_protobuf -lsgx_tstdc -lsgx_tcxx -lsgx_tcrypto -lsgx_tprotected_fs -l${SGX_TSVC_LIB} -Wl,--end-group \
        -Wl,-Bstatic -Wl,-Bsymbolic -Wl,--no-undefined \
        -Wl,-pie,-eenclave_entry -Wl,--export-dynamic \
        -Wl,--defsym,__ImageBase=0"
        )


include("cmake/setup_installation.cmake")