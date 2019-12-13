The Rest DSL is a facade that builds Rest endpoints as consumers for Camel routes , it can transform existing service to rest endPoints

 

Given OrderService with some implementation  DummyOrderService, the order service define 3 method :getOrder, updateOrder , createOrder and cancelOrder :

```JAVA
public interface OrderService {

    Order getOrder(int orderId);

    void updateOrder(Order order);

    String createOrder(Order order);

    void cancelOrder(int orderId);

}
```

## Define Camel context: 

1. Extend RouteBuilder.

1. Define the api entry point “/orders”.

1. Use rest verbs get, post, put delete ..

1. “To” the method that will be executed in the service.  bean:orderService?method=createOrder (where the bean is the serviceName, and the method is the method that will be executed.

```JAVA
import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;


@Component
public class OrderRoute extends RouteBuilder {

    @Override
    public void configure() throws Exception {
        // see the application.properties file for setup of the rest configuration

        // rest services under the orders context-path
        rest("/orders")
            // need to specify the POJO types the binding is using (otherwise json binding defaults to Map based)
            .get("{id}").outType(Order.class)
                .to("bean:orderService?method=getOrder(${header.id})")
                // need to specify the POJO types the binding is using (otherwise json binding defaults to Map based)
            .post().type(Order.class)
                .to("bean:orderService?method=createOrder")
                // need to specify the POJO types the binding is using (otherwise json binding defaults to Map based)
            .put().type(Order.class)
                .to("bean:orderService?method=updateOrder")
            .delete("{id}")
                .to("bean:orderService?method=cancelOrder(${header.id})");
    }
}

``` 

Or with ( XML DSL)

```XML
<camelContext id="camel"xmlns="http://camel.apache.org/schema/spring">
<rest path="/orders">
<get uri="{id}"><to uri="bean:orderService?method=getOrder(${header.id})"/>
</get>
<post>
<to uri="bean:orderService?method=createOrder"/>
</post><put><to uri="bean:orderService?method=updateOrder"/>
</put>
<delete uri="{id}">
<to uri="bean:orderService?method=cancelOrder(${header.id})"/>
</delete>
</rest>
</camelContext>
```