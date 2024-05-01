# All OSs Pyinstaller Github Action
This is a Github Action that creates executables for Linux, MacOS and Windows on a single release using Pyinstaller.

I've created this myself because searching on the web I couldn't find a similar solution. I hope this can be useful to someone else.

Feel free to contribute to this project by opening an issue or a pull request. If you find it useful please leave a :star: :blush: .

## :bulb: How it works
### Create release
Initially, we establish that the specified tasks should execute upon each push event occurrence, contingent upon the commit being tagged with a prefix of "v". Among these tasks, the createrelease job is responsible for initiating a new release on GitHub. This process involves employing the create-release action in its primary step. An action comprises a set of commands and procedures designed to accomplish a particular objective. Both pre-defined and user-defined actions are available for addressing common objectives.

To prepare for the following tasks, we'll be uploading files as assets to the recently generated release on GitHub. This process requires utilizing a specific URL location. This is accomplished in the initial step, which retrieves this location by accessing the content of the environment variable steps.create_release.outputs.upload_url and subsequently writes it to the file release_url.txt. Finally, we upload this text file, once more employing a pre-defined action, thereby enabling us to extract the URL from it at a later stage.

### Build
Next, we look at the build job. This is only run after 'createrelease' has finished. Here, it gets a little bit more tricky. Inside our strategy part, we define three different “versions” of a step. The first one will be run on MacOS. It will execute pyinstaller to build the MacOS package of our script. The second one will be run on a Windows machine and again use pyinstaller to create the Windows .exe version of the app. And the last one will be run on a Linux machine to create the Linux version.
```yml
- os: macos-latest
    TARGET: macos
    CMD_BUILD: >
        pyinstaller -F -w -n APP_NAME-Mac-x64 -i resources/icon.icns main.py &&
        cd dist/ &&
        zip -r9 APP_NAME-Mac-x64 APP_NAME-Mac-x64.app/                
    OUT_FILE_NAME: APP_NAME-Mac-x64.zip
    ASSET_MIME: application/zip
    - os: windows-latest
    TARGET: windows
    CMD_BUILD: pyinstaller -F -w -n APP_NAME-${{ github.ref_name }}-Win-x64 -i resources/icon.ico main.py
    OUT_FILE_NAME: APP_NAME-${{ github.ref_name }}-Win-x64.exe
    ASSET_MIME: application/vnd.microsoft.portable-executable
    - os: ubuntu-latest
    TARGET: linux
    CMD_BUILD: pyinstaller -F -n APP_NAME-${{ github.ref_name }}-Linux-x64 -i resources/icon.ico main.py
    OUT_FILE_NAME: APP_NAME-${{ github.ref_name }}-Linux-x64
    ASSET_MIME: application/zip 
```
After defining this strategy, the following steps are run:
- Using actions/checkout@v1, the repo is cloned to the cloud machine
- Python is installed
- The requirements.txt in the repo is used to resolve the dependencies of our app
- The packages are created using PyInstaller, according to the strategy defined beforehand (this step is run once for each TARGET)
- The resulting package is uploaded as an asset to the release

## :interrobang: How to use
### Create the correct directory structure
Create on the root directory a folder called '.github' and inside it another folder called 'workflows'. Inside the 'workflows' folder create a file called 'build.yml' with the content you can find on this repository.
```bash
mkdir -p .github/workflows/
touch .github/workflows/build.yml
```

### Replace the placeholders on the build.yml file
Replace the placeholders with your own values.
```txt
APP_NAME -> 'your_app_name'
main.py -> 'your_main_file.py'
```
Remember to check the correctness of the Python version you wnat to use and the proper existence of the 'requirements.txt' file.

### Push the release
Create a new push to Github with the following tag convention and the action will start automatically.
Replace '1.0' with your version.
```bash
git add .         
git commit -m "v1.0"  
git tag -a v1.0 -m "Version 1.0"           
git push origin master --tags   
```      
Remember that tags are unique, so you can't use the same tag twice (ideally you should improve the version ex: v1.0 -> v1.1 -> ...).

The previous commands will trigger the action and you will see the build process on the Actions tab of your repository.
![build-porcess](/img/build-process.png)

### Download the executables
After the build process is finished you can download the executables from the release page.
![main page release](/img/main-page-release.png)
![release](/img/release.png)

## :white_check_mark: Examples of usage
[Serial to UDP translator by Squadracorsepolito](https://github.com/squadracorsepolito/Serial-to-UDP-translator)

[SCanner Adapter by Squadracorsepolito](https://github.com/squadracorsepolito/SCanner-adapter)
