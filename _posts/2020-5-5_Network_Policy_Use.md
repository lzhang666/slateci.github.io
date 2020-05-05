&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;For sites that want to be extra sure that their systems are secure, Kubernetes presents multiple tools that can be used. One such tool is the Network Policy, a Kubernetes object that can be used to add an additional layer of security to a given service. It does so by allowing a user to specify what namespaces or CIDRs are allowed to access this service. SLATE now supports Network Policies in its application catalog, and activating these is a fairly simple process for the user.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;By default the applications in the catalog do not activate any network policies, and allow any individual from any site to access the application if the IP address and port is known. If this behaviour is not necessary for the app in question to function, then this can pose a security risk to the application itself and potentially the entire SLATE cluster the app is being run on.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;To activate the network policy of a given application some alterations to that app’s configuration must be made. To do this you will need to go onto the SLATE cluster you want to install the app on and type `slate app <app-name> get-conf`. This will give you the default configuration file for this application. Open this file in a text editor and go to the section that says `NetworkPolicy: false` and change the setting so that it is set to true. Next directly below there will be a list called AllowedCIDRs populated by dummy CIDRs. Place each of the CIDRs you want this app to be accessible to here, in the exact format that the dummy CIDRs currently are, making sure the spacing and - are placed the same as the example for each of your CIDRs.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Finally save your changes and deploy the application using your custom configuration. To do this give the install command `slate app install <application name> cluster=<cluster name> group=<group name> conf=<name of your custom config file>`. This will deploy the application in much the same way it normally would, but with the addition of a Network Policy that blocks traffic from any IP range not in the whitelisted CIDRs specified in the configuration document. It does so by simply dropping all packets from or to those disallowed IP ranges.