# gltf
Parses a single header file for gltf, depends on the json library (https://github.com/nlohmann/json), compiles with c++11, runs on mac windows android, Deserialisation gltf format.

Constructed based on the rules provided by the official GLTF documentation (https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#introduction-general)

Typical example
```
#include <string>
#include <iostream>
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
template<typename T>
void ReadVector(const std::string &path, int beg, int count, std::vector<T> &contents)
{
	FILE *fp = fopen(path.c_str(), "rb");
	if (fp)
	{
		fseek(fp, beg, SEEK_SET);
		contents.resize(count);
		fread(&contents[0], sizeof(T), count, fp);
		fclose(fp);
		return;
	}
	throw(errno);
}
struct vec3
{
    float x, y, z;
};
int main()
{
    std::string s;
    std::string path = "mesh/scene.gltf";
    ReadFile(path, s);
    // Deserialization
    gltf::glTF tf = gltf::Parse(s);

    std::cout << tf.asset.version << std::endl;
    
    size_t found = path.find_last_of("/\\");
    std::string dir = path.substr(0, found != std::string::npos ? found + 1 : 0);

    if (tf.meshes.size())
    {
        gltf::Mesh mesh = tf.meshes[0];
        for (const auto& attr : mesh.primitives[0].attributes)
        {
            if (attr.first == "POSITION")
            {
                std::vector<vec3> position;
                gltf::AccessResult result = gltf::Access(tf, attr.second);
                ReadVector(dir+result.buffer->uri, result.accessor->byteOffset+result.bufferView->byteOffset, result.accessor->count, position);
                for (const vec3& v : position)
                {
                    std::cout << "x: " << v.x << " y: " << v.y << " z: " << v.z << std::endl;
                }
                
            }
        }
    }
    return 0;
}
```
