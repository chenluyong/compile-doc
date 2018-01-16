# Bitcoin 签名编译脚本

```
hack.o: hack.cpp
    $(AM_V_CXX)$(CXX) $(DEFS) $(DEFAULT_INCLUDES) $(INCLUDES) $(libbitcoin_server_a_CPPFLAGS) $(CPPFLAGS) $(libbitcoin_server_a_CXXFLAGS) $(CXXFLAGS) -MT hack.o -MD -MP -MF $(DEPDIR)/hack.Tpo -c -o hack.o hack.cpp
    $(AM_V_at)$(am__mv) $(DEPDIR)/hack.Tpo $(DEPDIR)/hack.Po

example.o: example.cpp
    $(AM_V_CXX)$(CXX) $(DEFS) $(DEFAULT_INCLUDES) $(INCLUDES) $(bitcoind_CPPFLAGS) $(CPPFLAGS) $(bitcoind_CXXFLAGS) $(CXXFLAGS) -MT example.o -MD -MP -MF $(DEPDIR)/example.Tpo -c -o example.o $<
    $(AM_V_at)$(am__mv) $(DEPDIR)/example.Tpo $(DEPDIR)/example.Po

example.exe: hack.o example.o
    $(AM_V_CXXLD)$(bitcoind_LINK) example.o hack.o $(bitcoind_LDADD) $(LIBS)
```



example.cpp

```
extern void goodjob(const char* const path, const char* const _lastTransaction,
                const char* const _crtTransaction,
                const char* const _prvKey, const char* const _coinType);
int main(int argc, char* argv[]) {
	if (argc < 5) {
		return -1;2
	}
	std::cout << "bepal sign begin..." << std::endl;
	goodjob(argv[0],argv[1],argv[2],argv[3],argv[4]);
	std::cout << "bepal sign end..." << std::endl;
	return 0;
	}
```



