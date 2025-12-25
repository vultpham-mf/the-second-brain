# Authentication
<primary-label ref="draft"/>

<note>
    <p>Authentication basically answers the question: Who are you?</p>
    <ul>
        <li>If you are authenticated, you are allowed to access the system.</li>
        <li>When you perform an action, you might leave a trace of who performed it.</li>
        <li>In .NET, authentication is responsible for providing ClaimsPrincipal for authorization to make permissions decisions.</li>
    </ul>
</note>

## Examples & Overview

<a href="https://learn.microsoft.com/en-us/aspnet/core/security/authentication">Microsoft: Overview of ASP.NET Core authentication</a>

### Example: Registering authentication scheme
<code-block lang="c#">
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme,
            options => builder.Configuration.Bind("JwtSettings", options))
        .AddCookie(CookieAuthenticationDefaults.AuthenticationScheme,
            options => builder.Configuration.Bind("CookieSettings", options));

    {...}
    
    app.UseAuthentication();
    app.UseAuthorization();
</code-block>

- There are two authentication schemes (methods): <code>Cookie</code> and <code>JwtBearer</code>.
- The default scheme is <code>JwtBearer</code> because it is registered as default by this line of code:
    - <code>services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)</code>
- The Authentication middleware is added by calling <code>app.UseAuthentication()</code>
    - Call <code>app.UseAuthorization()</code> before any middleware that depends on users being authenticated.

### Example: Get user id
// TODO

## Concepts

### Authentication Scheme

- A scheme is basically an authentication method.
- An authentication scheme corresponds to:
    - An authentication handler.
    - Options for configuring that specific instance of the handler.

#### DefaultScheme

- When there is only a single authentication scheme registered, it is the default scheme
- To disable automatically using the single authentication scheme as the DefaultScheme, call <code>AppContext.SetSwitch("Microsoft.AspNetCore.Authentication.SuppressAutoDefaultScheme")</code>.

### Authentication Handler

- Classes:
    - RemoteAuthenticationHandler&lt;TOptions&gt;
    - AuthenticationHandler&lt;TOptions&gt;
- Handle logic for authenticating a user.
- Is derived from <code>AuthenticationHandler&lt;TOptions&gt;</code> or <code>IAuthenticationHandler</code>.
- It constructs an instance of <code>AuthenticationTicket</code>, representing the authenticated user.

### AuthenticationTicket
// TODO

### AuthenticationResult
The authentication scheme's authenticate action constructs the user's identity from the request context and returns an AuthenticateResult. This result indicates success and, if successful, provides the user's identity via an AuthenticationTicket. See AuthenticateAsync.

Authenticate examples include:
- A cookie authentication scheme constructing the user's identity from cookies.
- A JWT bearer scheme deserializing and validating a JWT bearer token to construct the user's identity.

### ClaimsPrincipal
// TODO

### Challenge

- An authentication challenge is invoked by Authorization when unauthenticated users attempt to access a resource that requires authorization.
- Examples:
    - A cookie authentication scheme redirecting the user to a login page. 
    - A JWT bearer scheme returning a 401 result with a <code>www-authenticate: bearer</code> header.

<note>A challenge action should let the user know what authentication mechanism to use to access the requested resource.</note>


