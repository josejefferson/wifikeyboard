# WifiKeyboard

## Gera um APK debug
```bash
./gradlew assembleDebug
```

## Gera um APK release
```bash
# Gera o APK
./gradlew assembleRelease

# Gera a chave de assinatura, se não tiver sido gerada ainda
# Somente utilize letras e números na senha!
keytool -genkey -v -keystore key.keystore -alias app-key -keyalg RSA -keysize 2048 -validity 10000

# Assina o APK digitalmente
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore key.keystore app/build/outputs/apk/release/app-release-unsigned.apk app-key

# Gera o base64 da sua chave
openssl base64 -A -in key.keystore -out key.keystore.base64.txt
```

## GitHub Actions
* Entre nas configurações do seu projeto > Secrets and variables > Actions
* Crie 4 secrets:
  * `ALIAS`: Nome da sua chave (padrão: `app-key`)
  * `KEYPASSWORD`: Senha que você criou ao gerar a chave de assinatura
  * `KEYSTOREPASSWORD`: Senha que você criou ao gerar a chave de assinatura
  * `SIGNINGKEYBASE64`: Base64 gerado da chave
---
* No arquivo `.github/workflows/build.yml`, altere as linhas destacadas com o nome do seu app
```yaml
      - name: Upload APK
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ github.token }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ${{steps.sign_apk.outputs.signedReleaseFile}}
          asset_name: WifiKeyboard ${{ steps.var.outputs.tag }}.apk
          #           ^^^^^^^^
          asset_content_type: application/zip

      - name: Upload AAB
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ github.token }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ${{steps.sign_aab.outputs.signedReleaseFile}}
          asset_name: WifiKeyboard ${{ steps.var.outputs.tag }}.aab
          #           ^^^^^^^^
          asset_content_type: application/zip
```

## Fazer a release no GitHub Actions
```bash
git tag v1.0.0
git push --tags
```