# escape=`

# Stage 0: prepare files
ARG BASE_IMAGE
FROM ${BASE_IMAGE} as builder

ARG COMMERCE_SIF_PACKAGE
ARG PSE_PACKAGE
ARG SXA_PACKAGE
ARG HABITAT_IMAGES_PACKAGE
ARG SCXA_PACKAGE
ARG SXA_STOREFRONT_PACKAGE
ARG SXA_STOREFRONT_THEME_PACKAGE
ARG SXA_STOREFRONT_CATALOG_PACKAGE

SHELL ["powershell", "-NoProfile", "-Command", "$ErrorActionPreference = 'Stop';"]

COPY files/ /Files/

RUN Expand-Archive -Path "/Files/$Env:COMMERCE_SIF_PACKAGE" -DestinationPath /Files/CommerceSIF -Force
COPY xc/sitecore/storefront/Install.SXA.Storefront.json /Files/CommerceSIF

COPY scripts/WebDeploy-Utilities.ps1 /Scripts/
# Remove DacFx and SQL dependencies from the WebDeploy (WDP) package which is not needed in a non-mssql container
RUN . /Scripts/WebDeploy-Utilities.ps1; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:PSE_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:SXA_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:HABITAT_IMAGES_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:SCXA_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:SXA_STOREFRONT_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:SXA_STOREFRONT_THEME_PACKAGE"; `
    RemoveDbDacFxDependencies -WdpFullpath "/Files/$Env:SXA_STOREFRONT_CATALOG_PACKAGE";

# Stage 1: perform actual build
FROM ${BASE_IMAGE}

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ARG PSE_PACKAGE
ARG SXA_PACKAGE
ARG HABITAT_IMAGES_PACKAGE
ARG SCXA_PACKAGE
ARG SXA_STOREFRONT_PACKAGE
ARG SXA_STOREFRONT_THEME_PACKAGE
ARG SXA_STOREFRONT_CATALOG_PACKAGE
ARG SITE_NAME="sitecore"
ARG WEB_TRANSFORM_TOOL

COPY --from=builder /Files/CommerceSIF /Files/CommerceSIF/
COPY --from=builder /Files/${PSE_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_PACKAGE} /Files/
COPY --from=builder /Files/${HABITAT_IMAGES_PACKAGE} /Files/
COPY --from=builder /Files/${SCXA_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_THEME_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_CATALOG_PACKAGE} /Files/
COPY --from=builder /Files/${WEB_TRANSFORM_TOOL} /Files/Microsoft.Web.XmlTransform.dll

# Install SXA Storefront
RUN [Environment]::SetEnvironmentVariable('PSModulePath', $env:PSModulePath + ';/Files/CommerceSIF/Modules'); `
    $sifConfigFile = Join-Path /files/ 'CommerceSIF/Install.SXA.Storefront.json'; `
    cd '/Files/CommerceSIF'; `
    Install-SitecoreConfiguration -Path $sifConfigFile `
    -InstallDir 'c:\\inetpub\\wwwroot\\sitecore' `
    -SiteName $Env:SITE_NAME `
    -MergeToolFullPath '/Files/Microsoft.Web.XmlTransform.dll' `
    -PowershellExtensionsWdpFullPath "/Files/$Env:PSE_PACKAGE"  `
    -SXAFrameworkWdpFullPath "/Files/$Env:SXA_PACKAGE" `
    -HabitatImagesWdpFullPath "/Files/$Env:HABITAT_IMAGES_PACKAGE" `
    -SXACommerceWdpFullPath "/Files/$Env:SCXA_PACKAGE" `
    -SXAStorefrontWdpFullPath "/Files/$Env:SXA_STOREFRONT_PACKAGE" `
    -SXAStorefrontThemeWdpFullPath "/Files/$Env:SXA_STOREFRONT_THEME_PACKAGE" `
    -SXAStorefrontCatalogWdpFullPath "/Files/$Env:SXA_STOREFRONT_CATALOG_PACKAGE"

ENTRYPOINT ["powershell", "C:/Boot.ps1"]