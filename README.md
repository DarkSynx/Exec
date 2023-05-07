# Exec
Permet de lancer un executable avec des parametres dans un fichier .ini; Allows you to launch an executable with parameters
![image](https://user-images.githubusercontent.com/9467611/236698176-61dd1f91-83ac-4536-bff5-a643b0cd29c0.png)
![image](https://user-images.githubusercontent.com/9467611/236698185-bd2f4a1c-577c-41c1-81a5-e3bb019003df.png)
```c++
#include <iostream>
#include <fstream>
#include <string>
#include <cstdlib>

#ifdef _WIN32
#include <windows.h>
#include <shellapi.h>
#endif

std::string get_executable_name() {
    #ifdef _WIN32
    TCHAR exe_path[MAX_PATH];
    GetModuleFileName(NULL, exe_path, MAX_PATH);
    std::string exe_name = exe_path;
    return exe_name.substr(0, exe_name.find_last_of('.')) + ".ini";
    #else
    return "";
    #endif
}

void create_default_ini(const std::string& ini_file) {
    std::ofstream file(ini_file);

    if (file.is_open()) {
        file << "run = \"\"" << std::endl;
        file << "param = \"\"" << std::endl;
        file << "debug = false" << std::endl;
        file.close();
    } else {
        std::cerr << "Unable to create default INI file: " << ini_file << std::endl;
    }
}

void read_ini(const std::string& ini_file, std::string& run_value, std::string& param_value, bool& debug) {
    std::ifstream file(ini_file);
    std::string line;

    if (file.is_open()) {
        while (std::getline(file, line)) {
            if (line.find("run = ") != std::string::npos) {
                run_value = line.substr(line.find("run = ") + 6);
            }
            if (line.find("param = ") != std::string::npos) {
                param_value = line.substr(line.find("param = ") + 8);
            }
            if (line.find("debug = ") != std::string::npos) {
                std::string debug_value = line.substr(line.find("debug = ") + 8);
                debug = (debug_value == "true");
            }
        }
        file.close();
    } else {
        std::cerr << "Unable to open INI file: " << ini_file << std::endl;
        create_default_ini(ini_file);
    }
}

int main() {
    std::string ini_file = get_executable_name();
    std::string run_value, param_value;
    bool debug = false;
    read_ini(ini_file, run_value, param_value, debug);

    std::ofstream debug_file;
    std::streambuf* original_cout_buffer = nullptr;

    if (debug) {
        debug_file.open("debug.txt");
        original_cout_buffer = std::cout.rdbuf();
        std::cout.rdbuf(debug_file.rdbuf());
    }

    if (!run_value.empty()) {
        std::cout << "- Lancement de " << run_value << " avec les parametres: " << param_value << std::endl;
        #ifdef _WIN32
        // Lancement de l'application avec les parametres
        ShellExecute(NULL, "open", run_value.c_str(), param_value.c_str(), NULL, SW_SHOWNORMAL);
        #endif
    } else {
        std::cerr << "Aucune application à lancer. Vérifiez le fichier INI." << std::endl;
    }

if (debug) {
    debug_file.flush();
    std::cout.rdbuf(original_cout_buffer);
    debug_file.close();
}

    return 0;
}
```
