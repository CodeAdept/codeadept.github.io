    Title: Unit Testing JSP Custom Tags
    Date: 2008-01-25T00:00:00
    Tags: java, jsp, tdd, spring

Testing J2EE components has always been a difficult task, which is probably why I see so many web projects that have few tests written for the web layer or sometimes none at all. Late last year Spring announced the release of Spring 2.5 , with some nice additions to the suite of mock testing objects for unit testing web components. That’s right unit testing web components, not in container testing. So like any good agile programmer let’s start with the test first.

<!-- more -->

``` java
public class SomeCustomTagTest extends TestCase {

    private SomeServiceInterface mockService;
    private SomeCustomTag someCustomTag;
    private MockServletContext mockServletContext;
    private MockPageContext mockPageContext;
    private WebApplicationContext mockWebApplicationContext;

    protected void setUp() throws Exception {
        super.setUp();
        // Create the mock servlet context
        mockServletContext = new MockServletContext();

        // Create the mock Spring Context so that we can mock out the calls to getBean in the custom tag
        // Then add the Spring Context to the Servlet Context
        mockWebApplicationContext = createMock(WebApplicationContext.class);
        mockServletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE,
                                        mockWebApplicationContext);

        // Create the MockPageContext passing in the mock servlet context created above
        mockPageContext = new MockPageContext(mockServletContext);

        // Create the mock service object using it's interface
        mockService = createMock(SomeServiceInterface.class);

        // Create an instance of the custom tag we want to test
        // set it's PageContext to the MockPageContext we created above
        someCustomTag = new SomeCustomTag();
        someCustomTag.setPageContext(mockPageContext);

        // Whenever you make a call to the doStartTag() method on the custom tag it calls getServletContext()
        // on the WebApplicationContext.  So to avoid having to put this expect statement in every test
        // I've included it in the setUp()
        expect(mockWebApplicationContext.getServletContext()).andReturn(mockServletContext).anyTimes();
    }

}
```
Some things you’ll notice in the setUp() method are that we’re creating a mock for the Spring Context. Since I don’t know of any way to have Spring inject dependencies into the custom tag, we’ll be getting a WebApplicationContext object from the container to get our beans from Spring. This mock allows us to fake calls to getBean() and return our mock service objects so we can unit test the custom tag without relying on external dependencies.
Now in order to get this test to compile we need to create the custom tag class. You’ll notice that we’re extending the RequestContextAwareTag class instead of TagSupport. The RequestContextAwareTag class is a Spring helper class that gives us some convenience methods for getting the WebApplicationContext among other things. You’ll also see that the method we need to implement is called doStartTagInternal() instead of doStartTag().

``` java
public class SomeCustomTag extends RequestContextAwareTag {

    protected int doStartTagInternal() throws Exception {

        // put some business logic here

        return SKIP_BODY;
    }

}
```

Now let’s write a failing test. Back in the unit test lets add a test method and a couple of helper methods for the mock objects.

``` java
public void testDoStartTag() throws Exception{
    String param = "Jeremy";
    String expectedOutput = "Hello, " + param;

    replayAllMocks();

    someCustomTag.setUserName(param);
    int tagReturnValue = someCustomTag.doStartTag();
    String output = ((MockHttpServletResponse) mockPageContext.getResponse()).getContentAsString();

    assertEquals("Tag should return 'SKIP_BODY'", TagSupport.SKIP_BODY, tagReturnValue);
    assertEquals("Output should be 'Hello, Jeremy'", expectedOutput, output);

    verifyAllMocks();
}

private void replayAllMocks() {
    replay(mockWebApplicationContext);
    replay(mockCompanyQuery);
}

private void verifyAllMocks() {
    verify(mockWebApplicationContext);
    verify(mockCompanyQuery);
}
```

In order to get the test to compile, you’ll need to modify the SomeCustomTag class to have a setter method for a userName parameter. We would like our custom tag to say “Hello” to a username that is passed in, so let’s implement the behavior we are trying to accomplish.

``` java
public class SomeCustomTag extends RequestContextAwareTag {

    private String userName;

    // A setter for an attribute that will be set in the custom tag
    public void userName(String userName) {
        this.userName = userName;
    }

    protected int doStartTagInternal() throws Exception {

        pageContext.getOut().print("Hello, " + userName);

        return SKIP_BODY;
    }

}
```

Next we would like to get the users full name from our Spring service. So in order to do that we need to tell the mockWebApplicationContext to return a mock of our service so we can unit test this in isolation. We also need to tell our mock service that we expect a call to getFullName() and what to return from this call. So modify our test method to resemble the following.

``` java
public void testDoStartTag() throws Exception{
    String param = "Jeremy";
    String fullName = "Jeremy Anderson";
    String expectedOutput = "Hello, " + fullName;

    expect(mockWebApplicationContext.getBean("someService")).andReturn(mockService);
    expect(someService.getFullName(param)).andReturn(fullName);
    replayAllMocks();

    someCustomTag.setUserName(param);
    int tagReturnValue = someCustomTag.doStartTag();
    String output = ((MockHttpServletResponse) mockPageContext.getResponse()).getContentAsString();

    assertEquals("Tag should return 'SKIP_BODY'", TagSupport.SKIP_BODY, tagReturnValue);
    assertEquals("Output should be 'Hello, Jeremy Anderson'", expectedOutput, output);

    verifyAllMocks();
}
```

Now when you run this code, you’ll get an error from EasyMock complaining about methods that were expected to be called, but weren’t. So lets fix that now. Modify the doStartTagInternal() method to look like the following.

``` java
protected int doStartTagInternal() throws Exception {
    // Get the Spring Context from the RequestContext
    WebApplicationContext context = getRequestContext().getWebApplicationContext();

    // Get the service we want to call from the Spring Context
    someService = (SomeServiceInterface) context.getBean("someService");

    String fullName = someService.getFullName(userName);
    pageContext.getOut().print("Hello, " + fullName);

    return SKIP_BODY;
}
```

Congratulations, you’ve now implemented a JSP Custom Tag using “Test First”.
