
if (NOT CMAKE_INSTALL_PREFIX OR
CMAKE_INSTALL_PREFIX STREQUAL "/usr/local")
set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_SOURCE_DIR}/../../../Install")
endif()


file(GLOB_RECURSE LITECOIN_SOURCE ${CMAKE_CURRENT_SOURCE_DIR} *.c *.h *.cpp)

add_custom_target(litecoin_dev ALL
DEPENDS ${CMAKE_INSTALL_PREFIX}/plugins/liblitecoind.so
)


add_custom_command(OUTPUT ${CMAKE_INSTALL_PREFIX}/plugins/liblitecoind.so
DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/src/litecoind
DEPENDS openssl_1.1.0h
COMMAND sh -x ../libtool --silent --tag=CXX --mode=link g++ -Wstack-protector -fstack-protector-all -fPIC -g -O2 -Wall -Wextra -Wformat -Wformat-security -Wno-unused-parameter -pthread -fPIC -shared -rdynamic -Wl,-z,relro -Wl,-z,now -pie -L/${CMAKE_CURRENT_SOURCE_DIR}/depends/x86_64-unknown-linux-gnu/share/../lib -Wl,-rpath=/${CMAKE_INSTALL_PREFIX}/openssl_1.1.0h/lib -Wl,-rpath=/${CMAKE_INSTALL_PREFIX}/bdb_4.8.30/lib -Wl,-rpath=/${CMAKE_INSTALL_PREFIX}/lib -o liblitecoind.so litecoind-bitcoind.o -L/${CMAKE_INSTALL_PREFIX}/openssl_1.1.0h/lib  -L/${CMAKE_INSTALL_PREFIX}/bdb_4.8.30/lib libbitcoin_server.a libbitcoin_common.a univalue/libunivalue.la libbitcoin_util.a libbitcoin_wallet.a libbitcoin_consensus.a crypto/libbitcoin_crypto.a leveldb/libleveldb.a leveldb/libmemenv.a secp256k1/libsecp256k1.la -L/usr/lib/x86_64-linux-gnu -lboost_system -lboost_filesystem -lboost_program_options -lboost_thread -lboost_chrono -ldb_cxx-4.8 -lssl -lcrypto -levent_pthreads -levent -levent
COMMAND ${CMAKE_COMMAND} -E copy liblitecoind.so ${CMAKE_INSTALL_PREFIX}/plugins/liblitecoind.so
COMMAND ${CMAKE_COMMAND} -E copy liblitecoind.so ${CMAKE_INSTALL_PREFIX}/../tests/regtest/shadow-litecoin/1_peerConnection/liblitecoind.so
COMMAND ${CMAKE_COMMAND} -E copy liblitecoind.so ${CMAKE_INSTALL_PREFIX}/../tests/regtest/shadow-litecoin/2_wallet/liblitecoind.so
WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/src
)

add_custom_command(OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/src/litecoind
DEPENDS bdb_4.8.30
DEPENDS ${LITECOIN_SOURCE}
#DEPENDS SHADOW_INTERFACE
COMMAND ./autogen.sh
COMMAND ./configure CPPFLAGS='-I${CMAKE_INSTALL_PREFIX}/bdb_4.8.30/include -I${CMAKE_INSTALL_PREFIX}/include' LDFLAGS='-L${CMAKE_INSTALL_PREFIX}/bdb_4.8.30/lib -L${CMAKE_INSTALL_PREFIX}/lib' --with-pic --disable-bench --disable-tests --enable-debug --disable-zmq --enable-wallet --with-incompatible-bdb
COMMAND make -j 6 
WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

# Define targets that can be used in same build-tree
add_library(litecoin_dev_::Plugin SHARED IMPORTED GLOBAL)
add_dependencies(litecoin_dev_::Plugin litecoin_dev)
set_target_properties(litecoin_dev_::Plugin PROPERTIES
IMPORTED_LINK_INTERFACE_LANGUAGES "C"
IMPORTED_LOCATION ${CMAKE_INSTALL_PREFIX}/plugins/liblitecoind.so)

