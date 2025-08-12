# RL Mera translate 

storePassword=CHANGE_ME_STORE_PASS
keyPassword=CHANGE_ME_KEY_PASS
keyAlias=rltranslator
storeFile=../rl_mera_translator.jks def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
    }

    android {
        ...
            signingConfigs {
                    release {
                                keyAlias keystoreProperties['keyAlias']
                                            keyPassword keystoreProperties['keyPassword']
                                                        storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
                                                                    storePassword keystoreProperties['storePassword']
                                                                            }
                                                                                }
                                                                                    buildTypes {
                                                                                            release {
                                                                                                        signingConfig signingConfigs.release
                                                                                                                    minifyEnabled false
                                                                                                                                shrinkResources false
                                                                                                                                            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
                                                                                                                                                    }
                                                                                                                                                        }
                                                                                                                                                            ...
                                                                                                                                                            } #!/usr/bin/env bash
                                                                                                                                                            set -e

                                                                                                                                                            # Usage:
                                                                                                                                                            # ./build_release.sh STORE_PASS KEY_PASS
                                                                                                                                                            # or set env vars:
                                                                                                                                                            # STORE_PASS=yourpass KEY_PASS=yourpass ./build_release.sh

                                                                                                                                                            STORE_PASS=${1:-$STORE_PASS}
                                                                                                                                                            KEY_PASS=${2:-$KEY_PASS}

                                                                                                                                                            if [ -z "$STORE_PASS" ] || [ -z "$KEY_PASS" ]; then
                                                                                                                                                              echo "Usage: ./build_release.sh STORE_PASS KEY_PASS"
                                                                                                                                                                echo "Or set env vars: STORE_PASS and KEY_PASS"
                                                                                                                                                                  exit 1
                                                                                                                                                                  fi

                                                                                                                                                                  JKS_FILE="../rl_mera_translator.jks"
                                                                                                                                                                  ALIAS="rltranslator"

                                                                                                                                                                  # Create keystore if not exists
                                                                                                                                                                  if [ ! -f "$JKS_FILE" ]; then
                                                                                                                                                                    echo "Creating keystore at $JKS_FILE ..."
                                                                                                                                                                      keytool -genkey -v -keystore "$JKS_FILE" -storetype JKS -keyalg RSA -keysize 2048 -validity 10000 -alias "$ALIAS" -storepass "$STORE_PASS" -keypass "$KEY_PASS" -dname "CN=RL Mera Translator, OU=Dev, O=You, L=City, S=State, C=IN"
                                                                                                                                                                      else
                                                                                                                                                                        echo "Keystore already exists at $JKS_FILE"
                                                                                                                                                                        fi

                                                                                                                                                                        # Create/overwrite key.properties
                                                                                                                                                                        cat > android/key.properties <<EOL
                                                                                                                                                                        storePassword=$STORE_PASS
                                                                                                                                                                        keyPassword=$KEY_PASS
                                                                                                                                                                        keyAlias=$ALIAS
                                                                                                                                                                        storeFile=$JKS_FILE
                                                                                                                                                                        EOL

                                                                                                                                                                        echo "Running flutter pub get..."
                                                                                                                                                                        flutter pub get

                                                                                                                                                                        echo "Building release APK..."
                                                                                                                                                                        flutter clean
                                                                                                                                                                        flutter build apk --release

                                                                                                                                                                        APK_PATH="build/app/outputs/flutter-apk/app-release.apk"
                                                                                                                                                                        if [ -f "$APK_PATH" ]; then
                                                                                                                                                                          echo "Release APK built successfully: $APK_PATH"
                                                                                                                                                                          else
                                                                                                                                                                            echo "Build finished but APK not found at expected path."
                                                                                                                                                                            fi