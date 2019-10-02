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

COPY scripts/Clear-Package.ps1 /Scripts/

# Prepare WebDeploy (WDP) package: Remove DacFx and SQL dependencies
RUN & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:PSE_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:SXA_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:HABITAT_IMAGES_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:SCXA_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:SXA_STOREFRONT_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:SXA_STOREFRONT_THEME_PACKAGE"; `
    & "/Scripts/Clear-Package.ps1" -Path "/Files/$Env:SXA_STOREFRONT_CATALOG_PACKAGE"; 

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
COPY --from=builder /Files/${WEB_TRANSFORM_TOOL} /Files/
COPY --from=builder /Files/${PSE_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_PACKAGE} /Files/
COPY --from=builder /Files/${HABITAT_IMAGES_PACKAGE} /Files/
COPY --from=builder /Files/${SCXA_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_THEME_PACKAGE} /Files/
COPY --from=builder /Files/${SXA_STOREFRONT_CATALOG_PACKAGE} /Files/

COPY scripts/install-package.json /Scripts/
COPY scripts/Invoke-XdtTransform.ps1 /Scripts/

# Install PSE Module
RUN Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:PSE_PACKAGE" `
    -SiteName $env:SITE_NAME

# Install SXA Module
RUN Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:SXA_PACKAGE" `
    -SiteName $env:SITE_NAME; `
    $env:SITE_PATH = 'c:/inetpub/wwwroot/{0}' -f $env:SITE_NAME; `
    $env:XDTDLL_PATH = Join-Path /files/ $env:WEB_TRANSFORM_TOOL; `
    $env:WEBCONFIG_PATH = Join-Path $env:SITE_PATH '\\Web.config'; `
    Write-Host $env:XDTDLL_PATH; `
    & "/Scripts/Invoke-XdtTransform.ps1" -Path $env:WEBCONFIG_PATH -XdtPath (Join-Path $env:SITE_PATH '\\App_Data\\SxaXdt\\Web.config.xdt') -XdtDllPath $env:XDTDLL_PATH;

# Install SCXA Module
RUN Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:SCXA_PACKAGE" `
    -SiteName $env:SITE_NAME

# Install SXA Storefront: SXA Storefront modules, SXAStorefrontPostInstallationSteps
RUN Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:HABITAT_IMAGES_PACKAGE" `
    -SiteName $env:SITE_NAME; `
    Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:SXA_STOREFRONT_PACKAGE" `
    -SiteName $env:SITE_NAME; `
    Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:SXA_STOREFRONT_THEME_PACKAGE" `
    -SiteName $env:SITE_NAME; `
    Install-SitecoreConfiguration -Path "/Scripts/install-package.json" `
    -Package "/Files/$Env:SXA_STOREFRONT_CATALOG_PACKAGE" `
    -SiteName $env:SITE_NAME; `
    # SXAStorefrontPostInstallationSteps
    [Environment]::SetEnvironmentVariable('PSModulePath', $env:PSModulePath + ';/Files/CommerceSIF/Modules'); `
    $env:SITE_PATH = 'c:/inetpub/wwwroot/{0}' -f $env:SITE_NAME; `
    Install-SitecoreConfiguration -Path '/Files/CommerceSIF/Configuration/Commerce/SXAStorefront/_SXAStorefront.Post.Installation.json' `
    -MergeToolFullPath (Join-Path /files/ $env:WEB_TRANSFORM_TOOL) `
    -InstallDir $env:SITE_PATH;


ENTRYPOINT ["powershell", "C:/Boot.ps1"]