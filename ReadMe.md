# Nginx-Lab


**This repository Nginx-Lab-cb677c81a2654db88631a8d10aa32a9a will move to Nginx-Lab when it is ready.**
























This is a web-application to lab with [Nginx](https://www.nginx.com). It's about configuring an deploying.

![.github/workflows/Docker-deploy.yml](https://github.com/HansKindberg-Lab/Nginx-Lab/actions/workflows/Docker-deploy.yml/badge.svg)

Web-application, without configuration, pushed to Docker Hub: https://hub.docker.com/r/hanskindberg/nginx-lab

## 1 Configuration

- [Example](/Source/Application/appsettings.Development.json)

## 2 Development

### 2.1 Nginx

This applies if you are on Windows.

Download [Nginx for Windows](http://nginx.org/en/docs/windows.html). Download the latest mainline version distribution, http://nginx.org/en/download.html. Eg. http://nginx.org/download/nginx-1.23.1.zip. Unzip it and copy the content to "C:\Program Files\nginx".

On my Windows, port 80 and 443 are already occupied by the system. You can check it by the following command:

	netstat -ano | find "80" | find "LISTEN"

If you get a result like this:

	TCP   0.0.0.0:80    0.0.0.0:0    LISTENING    4

Then port 80 is already occupied by process-id 4. To check which process is running it, run the following command:

	tasklist /fi "PID eq 4"

If port 80 is already occupied you need to change the nginx configuration before you start nginx. In C:\Program Files\nginx\conf\nginx.conf, change row 36 from:

	listen 80;

to

	listen 8080;

You may change it to what ever port you want.

Now you can start nginx by double-clicking "C:\Program Files\nginx\nginx.exe". Browse to http://localhost:8080 and you should see a Nginx welcome message. You can verify that Nginx is running by checking the Task Manager. A nginx.exe process should be running.

### 2.2 Hosts

This applies if you are on Windows.

To be able to run in Visual Studio on Windows you need to add the following to **C:\Windows\System32\drivers\etc\hosts**:

	127.0.0.1 nginx-lab.local
	127.0.0.1 mtls.nginx-lab.local







### 2.3 OpenShift

[Install Red Hat OpenShift 4 on your laptop](https://developer.ibm.com/blogs/install-openshift-4-on-your-laptop/)


[Install OpenShift 4 on a laptop with CodeReady Containers](https://cloud.redhat.com/openshift/install/crc/installer-provisioned)


https://github.com/code-ready/crc





===============================================================================

# Notes

[Tutorial: Create a multi-container app with Docker Compose](https://docs.microsoft.com/en-us/visualstudio/containers/tutorial-multicontainer)

===============================================================================







## Why

I want to deploy an [IdentityServer](https://github.com/DuendeSoftware/IdentityServer)-implementation to OpenShift (Kubernetes). The [IdentityServer](https://github.com/DuendeSoftware/IdentityServer)-solution needs to be able to handle client-certificate-authentication (mTLS).

Both the [IdentityServer](https://github.com/DuendeSoftware/IdentityServer)-way:

- [TLS Client Certificates](https://docs.duendesoftware.com/identityserver/v5/tokens/authentication/mtls/)

and interactively (as a user, in the browser, you choose to sign in with a certificate and you get a certificate-picker pop-up)

- https://github.com/HansKindberg/IdentityServer-Extensions/blob/master/Source/Implementations/Identity-Server/Application/Controllers/AuthenticateController.cs#L89

I have struggled a lot to find a solution for this. With Windows IIS I know how to do it, https://github.com/HansKindberg/IdentityServer-Extensions/blob/master/Source/Implementations/Identity-Server/Application/Web.config:

	<configuration>
		<location path="Authenticate/Certificate">
			<system.webServer>
				<handlers>
					<add modules="AspNetCoreModuleV2" name="aspNetCore" path="*" resourceType="Unspecified" verb="*" />
				</handlers>
				<security>
					<access sslFlags="Ssl, SslNegotiateCert, SslRequireCert" />
				</security>
			</system.webServer>
		</location>
		<location path="connect/mtls">
			<system.webServer>
				<handlers>
					<add modules="AspNetCoreModuleV2" name="aspNetCore" path="*" resourceType="Unspecified" verb="*" />
				</handlers>
				<security>
					<access sslFlags="Ssl, SslNegotiateCert, SslRequireCert" />
				</security>
			</system.webServer>
		</location>
	</configuration>

Path-based mTLS seems to be supported in Apache to.

The IdentityServer-people suggests NGINX as a good choice, using sub-domain for mTLS:

- https://leastprivilege.com/2020/02/07/mutual-tls-and-proof-of-possession-access-tokens-part-1-setup/

Then I asked a question about how to laborate with NGINX locally on Windows:

- https://github.com/DuendeSoftware/IdentityServer/discussions/823

They answered with a second opportunity, to do it with Kestrel, and then I started to think of this solution.

The idea is to configure the IdentityServer-application something like this:

	{
		"AllowedHosts": "id.example.org;mtls.id.example.org",
		"Kestrel": {
			"Endpoints": {
				"Default": {
					"Sni": {
						"mtls.id.example.org": {
							"ClientCertificateMode": "RequireCertificate"
						},
						"*": {
							"ClientCertificateMode": "NoCertificate"
						} 
					},
					"Url": "https://*:5000"
				}
			}
		}
		...
	}

Then you create two CNAME's in you DNS and point them to an IP handled by your OpenShift-cluster:

- id.example.org
- mtls.id.example.org

In OpenShift you have a route ................................... more to come

## How to configure Kestrel

Many examples are through code but I want to do it in appsettings.json.

- [Configure endpoints for the ASP.NET Core Kestrel web server](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel/endpoints)