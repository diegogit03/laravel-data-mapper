# Laravel Data Mapper

Um simples Data mapper para laravel.

## Mapeamento

No laravel data mapper é possivel tanto mapear com um simples objeto PHP:

```php
class Customer {
    public function __construct(
        public string $name,
        public string $email
    ) {}
}

class Order {
    public int $id;
    
    public Customer $customer;
    
    public Money $total;
}

final class OrderMapper extends EntityMapper {
    protected function map (MetadataMapper $mapper) {
        $mapper->mapField('customer')
            ->toRow(fn (Order $order) => [
                'customer_name' => $order->customer->name,
                'customer_email' => $order->customer->email 
            ])
            ->fromRow(fn (Object $row) => new Customer($row->customer_name, $customer->customer_email));
        $mapper->mapField('price')->fromRow(fn (object $row) => new Money($row->price, 'BRL'));
    }
}
```

Uso do Mapper:

```php
$mapper = new OrderMapper();

# Persiste entidade
$order = new Order();

$order->customer = new Customer('Teste', 'teste@email.com');
$order->total = new Money(100, 'BRL');

$mapper->persist($user);

$mapper->customer->name = 'Teste 2';

$mapper->persist($user);

# Recupera entidade
$order = $mapper->findBy('id', 1);

# Mapeando collections e arrays
$ordersCollection = DB::table('orders')->get();

$orders = $mapper->fromCollection($ordersCollection);
$orders = $mapper->fromArray($ordersCollection->toArray());
```

Uso dentro de um repository:

```php
class OrderRepository {
    function __construct(private OrderMapper $mapper) {}

    public function save(Order) {
        $this->mapper->perist($order);
    }
    
    /**
    * @return array<Order>
    */
    public function findAll(): array {
        $ordersCollection = DB::table('orders')->get();

        return $this->mapper->fromCollection($ordersCollection);
    }
    
    public function findById(int $id): ?Order {
        return $this->mapper->findBy('id', $id);
    }
}
```
