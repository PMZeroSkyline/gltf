# gltf
Parses a single header file for gltf, depends on the json library (https://github.com/nlohmann/json), compiles with c++11, runs on mac windows android

Constructed based on the rules provided by the official GLTF documentation (https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#introduction-general)

```
#include <string>
#include "gltf.h"

void ReadFile(const std::string &path, std::string &contents)
{
	FILE *fp = fopen(path.c_str(), "rb");
	if (fp)
	{
		fseek(fp, 0, SEEK_END);
		contents.resize(ftell(fp));
		rewind(fp);
		fread(&contents[0], 1, contents.size(), fp);
		fclose(fp);
		return;
	}
	throw(errno);
}
int main()
{
    std::string s;
    ReadFile("scene.json", s);
    gltf::glTF tf = gltf::Parse(s);

    return 0;
}
```