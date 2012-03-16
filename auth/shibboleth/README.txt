Shibboleth Authentication Plugin for Moodle
-------------------------------------------------------------------------------

Requirements:
- Shibboleth Service Provider 2.x
  See documentation for your Shibboleth federation on how to deploy the
  Service Provider.


How the Shibboleth authentication works
--------------------------------------------------------------------------------
To authenticat users with Shibboleth in Moodle a user must access the
Shibboleth-protected page /auth/shibboleth/index.php. If Shibboleth is the only
authentication method (see 4.a), this happens automatically when a user selects
his home organization in the Moodle WAYF service or if the alternate login URL
is configured to be the protected /auth/shibboleth/index.php
Otherwise, the user has to click on the link on the dual login page you
provided in step 5.b.

Moodle then checks whether the Shibboleth attribute that is mapped
as the username is present. This attribute should only be present if a user is
Shibboleth authenticated.

If the user's Moodle account has not existed yet, it gets automatically created.

To prevent that every Shibboleth user can access the Moodle site it is 
recommended to set proper access control rules and for example replace the line 
'require valid-user' of the Apache configuration (see above) with a more 
specific access control rule. The access control rules defined in step 1 are 
encorced by Shibboleth itself. Only users who met these rules can access 
/auth/shibboleth/index.php and get logged in.

It is possible to use Shibboleth AND another authentication method (it was tested with
manual login). If there are a few users that don't have a Shibboleth
login, manual accounts can be created for them. Users can log in only via one
authentication method unless they have multiple accounts in Moodle.


Moodle Configuration for Shibboleth
-------------------------------------------------------------------------------
1. Protect the directory moodle/auth/shibboleth/index.php with Shibboleth.
   The page index.php in that directory logs in a Shibboleth user and creates
   a Moodle session for this user.
   For Apache define a rule like the following in the Apache configuration:

--
<Directory  /path/to/moodle/auth/shibboleth/index.php>
        AuthType shibboleth
        ShibRequireSession On
        require valid-user
</Directory>
--

   To restrict access to Moodle, replace the access rule 'require valid-user'
   with something that fits your needs, e.g. 'require affiliation student'.

   For IIS protect the auth/shibboleth directory directly in the
   RequestMap of the Shibboleth configuration file (shibboleth2.xml).

--
<Path name="moodle" requireSession="false" >
   <Path name="auth/shibboleth/index.php" requireSession="true" >
      <AccessControl>
          ...
      </AccessControl>
   </Path>
</Path>
--

   Also see:
   https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPRequestMapper and
   https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPAccessControl

2. As Moodle admin, go to the 'Site Administration >> Plugins >> Authentication 
   >> Manage Authentication', enable Shibboleth authentication and then click on 
   the the 'Shibboleth' settings.

3. Fill in the fields of the form. The fields 'Username', 'First name',
   'Surname', etc. should contain the name of the environment variables of the
   Shibboleth attributes that shall be mapped onto the corresponding Moodle
   profile fields (check /Shibboleth.sso/Session after authentication to
   see a list of available Shibboleth attributes and their names).
   
   The mapping for the Moodle 'Username' field is of great importance because
   this attribute is used for the Moodle authentication of Shibboleth users.
   For Moodle to work properly Shibboleth needs to provide the attribute
   for the username. It must be unique and persistent. Note that moodle treats 
   this username as case-insensitive.
   
   All attributes used for moodle must obey a certain length, otherwise Moodle
   cuts off the ends. Consult the Moodle documentation for further information
   on the maximum lengths for each field in the user profile.

4.a  If Shibboleth shall be the only authentication method with an external
     Discovery Service/WAYF service , set the 'Alternate Login URL' in the
     'Manage Authentication' to the the URL of the file 
     'moodle/auth/shibboleth/index.php'. This will enforce Shibboleth login.

4.b If you want to use the Moodle integrated WAYF service, you have to activate
    it in the Moodle Shibboleth authentication settings by checking the
    'Moodle WAYF Service' checkbox and providing a list of entity IDs in the
    'Identity Providers' textarea together with a name and an optional
    SessionInitiator URL, which usually is a relative URL pointing
    to the same host. If no specific SessionInitiator URL is given, the default 
    one (/Shibboleth.sso/Login) will be used.
    Also see https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPSessionInitiator

    Important Note: If you upgraded from a previous version of Moodle and now
                    want to use the integrated WAYF, you have to make sure that
                    in step 1 only the index.php script in
                    moodle/auth/shibboleth/ is protected but *not* the other
                    scripts and especially not the login.php script.

5.  Save the changes for the 'Shibboleth settings'.

    Important Note: If you went for 4.b (integrated WAYF service), saving the
                    settings will overwrite the Moodle Alternate Login URL. 
                    Be sure that Shibboleth authentication works reliably 
                    with another user while still logged in as Moodle 
                    administrator. Otherwise, you could lock yourself out of 
                    Moodle.

6.  If you want to use Shibboleth in addition to another authentication method
     not using the integrated WAYF service from 4.b, change the 'Instructions' 
     text in of 'Site Administration >> Plugins >> Authentication >> 
     Manage authentication' to contain a link to the file
     moodle/auth/shibboleth/index.php which is protected by
     Shibboleth (see step 1.) and causes the Shibboleth login procedure to start.
     You can also use HTML code in that field, e.g. to include an image as a
     Shibboleth login button.

     Note: As of now it not possible to use dual login together with the integrated
           WAYF service provided by Moodle (4.b).

7. Save the authentication changes.


Shibboleth dual login with custom login page
--------------------------------------------------------------------------------
It is possible to create a dual login page that allows users to authenticate in
different ways. For this to work, it is necessary to set up the two 
authentication methods (e.g. 'Manual Accounts' and 'Shibboleth') and specify an 
alternate login link to an own dual login page. On that page a link to the 
Shibboleth-protected page ('/auth/shibboleth/index.php') must be placed for the 
Shibboleth login and a form that sends 'username' and 'password' to 
moodle/login/index.php. Set this web page then als alternate login page.
Consult the Moodle documentation for further instructions and requirements.


Process Shibboleth attributes before mapping to Moodle fields
--------------------------------------------------------------------------------
Among the Shibboleth settings in Moodle there is a field that should contain a
path to a php file that can be used as data manipulation hook.
This file allows to further process (change, split, combine, ...) Shibboleth 
attributes in order to create a Moodle user record.

Example 1: A Shibboleth federation uses an attribute that specifies the
           user's preferred language, but the content of this attribute is not
           compatible with the Moodle profile field. E.g. the Shibboleth
           attribute contains 'German' but Moodle needs a two letter value like
           'de'.

Example 2: The country, city and street are provided in one Shibboleth attribute
           but Moodle wants them split up in street, city and zip code.

In order to use this Data modification API one must be a skilled PHP programmer. 
It is strongly recommended to take a look at the file
moodle/auth/shibboleth/auth.php, especially the function 'get_userinfo'
where this Data modification API file is included. The context of the file is the 
same as within this login function. 
So the code should directly modify the $result array. This also works for
custom attributes. All field names of the keys in the $result array must be 
lowercase.

Example file:

--
<?PHP

    // Set the zip code and the adress
    if ($_SERVER[$this->config->field_map_address] != '')
    {
        // $address contains something like 'SWITCH$Limmatquai 138$CH-8021 Zurich'
        // We want to split this up to get:
        // institution, street, zipcode, city and country
        $address = $_SERVER[$this->config->field_map_address];
        list($institution, $street, $zip_city) = explode('$', $address);
        preg_match('/ (.+)/', $zip_city, $regs);
        $city = $regs[1];

        preg_match('/(.+)-/',$zip_city, $regs);
        $country = $regs[1];

        $result["address"] = $street;
        $result["city"] = $city;
        $result["country"] = $country;
        $result["department"] = $institution;
        $result["description"] = "I am a Shibboleth user";

    }

?>
--

Logout Support
--------------------------------------------------------------------------------
In order make Moodle support Shibboleth logout, the Shibboleth Service Provider 
has to be made aware of the Moodle logout capabilities. Only then the SP
can trigger Moodle's front or back channel logout handler.

To make the SP aware of the Moodle logout, add the following to the
Shibboleth main configuration file shibboleth2.xml (usually in /etc/shibboleth/)
just before the <MetadataProvider> element.

--
<Notify
    Channel="back"
    Location="https://#YOUR_MOODLE_HOSTNAME#/moodle/auth/shibboleth/logout.php" />
--

Then restart the Shibboleth daemon and check the log file for errors. If there
were no errors, test the logout feature by accessing Moodle,
authenticating via Shibboleth and the access the URL:
#YOUR_MOODLE_HOSTNAME#/Shibboleth.sso/Logout (assuming this is a standard
Shibboleth installation). If everything worked well, there should be a Shibboleth
page saying that you were successfully logged out. 

Requirements:
- PHP needs the Soap Extension, which must be installed manually for PHP < 5.0.1:
  More information is available here http://ch.php.net/soap
- Logout only works with Shibboleth Service Provider 2.1 or newer
- /moodle/auth/shibboleth/logout.php *must not* be protected by Shibboleth!
  In case all of Moodle is protected with Shibboleth, add directives
  like the following to the Apache configuration after all the other require 
  access control rules

--
<Directory /path/to/moodle/auth/shibboleth/logout.php>
    AuthType shibboleth
    ShibRequireSession Off
    require shibboleth
</Directory>
--
  When using IIS, the same can be achieved with something like:
--
<Path name="auth/shibboleth/logout.php" requireSession="false" >
--
  in the shibboleth2.xml RequestMap.


Limitations:
Single Logout is only supported when SAML2 is used at the SP and the IdP.
As of April 2012, the latest Shibboleth Identity Provider does not yet support
Single Logout (SLO). Therefore, the single logout feature cannot be used yet
when the user's Identity Provider is using Shibboleth. There are other SAML2 
implementations like SimpleSAML that support parts of the SLO bindings.
The easiest and safest way to log out still is to show the users a web page
that instructs them to quit their web browsers.

Also see https://wiki.shibboleth.net/confluence/display/SHIB2/SLOIssues and
https://wiki.shibboleth.net/confluence/display/SHIB2/NativeSPLogoutInitiator 
for some background information on the logout topic.


Changes
--------------------------------------------------------------------------------
- 11. 2004: Created by Markus Hagman
- 05. 2005: Modifications to login process by Martin Dougiamas
- 05. 2005: Various extensions and fixes by Lukas Haemmerle
- 06. 2005: Adaptions to new field locks and plugin config structures by Martin
            Langhoff and Lukas Haemmerle
- 10. 2005: Added better error messages and moved text to language directories
- 02. 2006: Simplified authentication so that authorization works properly
            Added instructions for IIS
- 11. 2006: User capabilities are now loaded properly as of Moodle 1.7+
- 03. 2007: Adapted authentication method to Moodle 1.8
- 07. 2007: Fixed a but that caused problems with uppercase usernames
- 10. 2007: Removed the requirement for email address, surname and given name
            attributes on request of Markus Hagman
- 11. 2007: Integrated WAYF Service in Moodle
- 12. 2008: Shibboleth 2.x and Single Logout support added
- 1.  2008: Added logout hook and moved Shibboleth config strings to utf8 auth
            language files.
- 3.  2009: Added various improvements and bug fixes reported by Ina Müller from
            university Tübingen and Peter Ellis of University of Washington
- 4.  2009: Added another requirement for logout regarding the call back script
- 6.  2009: Changed handler URL when integrated Discovery Service is used
- 10. 2009: Fixed HTML entity preservation in Shibboleth settings
- 03. 2012: Added support for custom attributes. Improved some lang strings.
