FROM mcr.microsoft.com/powershell:latest

# Labels for GitHub Actions
LABEL "com.github.actions.name"="ARM Template Toolkit"
LABEL "com.github.actions.description"="Test ARM template using the ARM Template Toolkit"

RUN apt-get update && \
	apt-get upgrade -y && \
	apt-get install -y git curl wget unzip

# Clone repo in local executions only
#RUN git clone --depth 1 https://github.com/Yvand/AzureRM-Templates.git /github/workspace

# Download arm-ttk
ENV ARMTTK_PATH="/src/arm-ttk/arm-ttk"
# RUN git clone --depth 1 https://github.com/Azure/arm-ttk.git $ARMTTK_PATH
RUN curl -s https://api.github.com/repos/Azure/arm-ttk/releases/latest | grep "browser_download_url.*arm-ttk.zip" | cut -d : -f 2,3 | tr -d \" | xargs wget --directory-prefix=/src -q
RUN unzip "/src/arm-ttk.zip" -d /src
RUN test -f "$ARMTTK_PATH/arm-ttk.psd1"

# Copy the Entrypoint file from the current directory (the one the Dockerfile is located in)
COPY entrypoint.ps1 /src/entrypoint.ps1

# Start using the Entrypoint file.
ENTRYPOINT [ "pwsh", "/src/entrypoint.ps1" ]