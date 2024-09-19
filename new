# Go C++ blank template generator
#
# Python script for setting up a blank C++ project.
# See https://gocpp.dev
#
# Copyright (C) 2024 Cemalettin Dervis

import sys
import os
import urllib.request
import shutil
import subprocess
import platform
from pathlib import Path

is_linux = sys.platform == 'linux' or sys.platform == 'linux2'
is_macos = sys.platform == 'darwin'
is_windows = sys.platform == 'win32'
machine = platform.machine()

server_url = 'https://gocpp.dev'
cmake_build_dir = 'build'

def configure_file(src_filename: str, dst_filename: str, placeholders, delete_src_file_afterwards = False):
    with open(src_filename, 'r') as file:
        contents = file.read()

    for (key, value) in placeholders:
        contents = contents.replace(key, value)

    with open(dst_filename, 'w') as file:
        file.write(contents)

    if delete_src_file_afterwards:
        os.remove(src_filename)

def download_from_server(src_filename: str, dst_filename: str):
    basename = os.path.basename(src_filename)
    src = src_filename.replace(basename, basename[1:]) if basename.startswith('.') else src_filename

    print('Fetch', src)

    urllib.request.urlretrieve(
        f'{server_url}/{src}',
        os.path.join(app_dir, dst_filename)
    )

# TODO: make this configurable some day
app_name = 'MyApp'

# Check prerequisite programs
git_cmd = shutil.which('git')
if git_cmd == None:
    sys.exit('error: unable to find git; please install git first.')

cmake_cmd = shutil.which('cmake')
if cmake_cmd == None:
    sys.exit('error: unable to find CMake; please install CMake first. https://cmake.org')

# Create app folder and sources.
app_dir = os.path.join(os.getcwd(), app_name)

if os.path.exists(app_dir):
    sys.exit(f'error: directory {app_dir} already exists')

# Create empty Git repository
subprocess.check_call([
    git_cmd, 'init', app_dir,
], shell=False)

app_cmake_dir = os.path.join(app_dir, 'cmake')
os.makedirs(app_cmake_dir)

app_src_dir = os.path.join(app_dir, 'src')
os.makedirs(app_src_dir)

# Download common files.
for file in ['.gitignore', '.clang-format', '.clang-tidy', '.clangd', 'CMakePresets.json']:
    download_from_server(f'blank_code/{file}', os.path.join(app_dir, file))

# Download CMake script files
for file in ['CompilerSetup.cmake', 'CPM.cmake']:
    download_from_server(f'blank_code/cmake/{file}', os.path.join(app_cmake_dir, file))

# Download source files.
for file in ['main.cpp']:
    download_from_server(f'blank_code/src/{file}', os.path.join(app_src_dir, file))

# Create CMakeLists.txt
with open(os.path.join(app_dir, 'CMakeLists.txt'), 'w') as file:
    dst_filename = os.path.join(app_dir, 'CMakeLists.txt.in')
    download_from_server('blank_code/CMakeLists.txt', dst_filename)

    configure_file(
        dst_filename, os.path.join(app_dir, 'CMakeLists.txt'),
        placeholders=[('${GOCPP_DEV_APP_NAME}', app_name)],
        delete_src_file_afterwards=True
    )

# Configure using CMake
subprocess.check_call([
    cmake_cmd, '--preset', 'dev',
], cwd=app_dir, shell=False)

subprocess.check_call([
    cmake_cmd, '--build', '--preset', 'dev',
], cwd=app_dir, shell=False)

editors_available: list[str] = []

for editor in ['clion', 'code', 'zed', 'subl', 'qtcreator']:
    location = shutil.which(editor)
    if location != None:
        editors_available.append(location)

editor_count = len(editors_available)
have_opened_app = False

print('')
print('--- Done ---')
print('')
print('Your app is ready at:', app_dir)

vs_sln_filename = os.path.join(app_dir, cmake_build_dir, f'{app_name}.sln')
if is_windows and os.path.exists(vs_sln_filename):
    print('Opening Visual Studio solution', vs_sln_filename)
    os.startfile(vs_sln_filename)
    have_opened_app = True

elif editor_count > 0:
    if editor_count == 1:
        print('I detected the following editor in your environment:', editors_available[0])

        choice = input('Shall I open the project with it? y/n: ')
        if choice == 'y':
            subprocess.call([editors_available[0], app_dir], shell=False)
            have_opened_app = True
    else:
        print('I detected the following editors in your environment:')

        index = 1
        for editor in editors_available:
            print(f'[{index}] {editor}')
            index += 1

        print('Shall I open the project with one of those?')
        choice = input(f'[1-{editor_count} or n]: ')
        if choice != 'n':
            index = int(choice)
            if index != None and (index-1) >= 0 and (index-1) < editor_count:
                subprocess.call([editors_available[index-1], app_dir], shell=False)
                have_opened_app = True

if not have_opened_app:
    instruction_msg = f"""
You can cd into the app directory, then ...

    Setup:       cmake --preset dev
    Build:       cmake --build --preset dev
    Run:         ./{cmake_build_dir}/{app_name}
    Add dependencies in vcpkg.json
    Link those dependencies in CMakeLists.txt
"""

    if os.path.exists(vs_sln_filename):
        instruction_msg += f'- Open the Visual Studio solution at: {vs_sln_filename}\n'

    if editor_count == 0:
        print(instruction_msg)
    else:
        print(f'Alright. {instruction_msg}')

    print('Have fun!')
