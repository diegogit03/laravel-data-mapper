# Eloquent Data Mapper

Utilize Data mapper no eloquent sem perder o conforto do laravel.

Funcionalidades:
- Data Mapping
- Unit of Work
- Identity Map
- Laravel First

## Data Mapping

No eloquent data mapper é possivel tanto mapear com um simples objeto PHP:

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
        $mapper->map('id')->id();
        $mapper->map('customer')
            ->toRow(fn (Order $order) => [
                'customer_name' => $order->customer->name,
                'customer_email' => $order->customer->email
            ])
            ->fromRow(fn (Object $row) => new Customer($row->customer_name, $customer->customer_email));
        $mapper->map('total')
            ->toRow(fn (Order $order) => [
                'total' => $order->total->toFloat()
            ])
            ->fromRow(fn (object $row) => new Money($row->price, 'BRL'));
    }
}
```

Uso do Mapper:

```php
$em = new EntityManager();

# Persiste entidade
$order = new Order();

$order->customer = new Customer('Teste', 'teste@email.com');
$order->total = new Money(100, 'BRL');

$em->orders->persist($user);

$order->customer->name = 'Teste 2';

$em->orders->persist($user);

# Recupera entidade
$order = $em->orders->findBy('id', 1);

# Mapeando collections e arrays
$ordersCollection = DB::table('orders')->get();

$orders = $em->orders->fromCollection($ordersCollection);
$orders = $em->orders->fromArray($ordersCollection->toArray());
```

Uso dentro de um repository:

```php
class OrderRepository {
    function __construct(private EntityManager $em) {}

    public function save(Order $order) {
        $this->em->orders->persist($order);
    }
    
    /**
    * @return array<Order>
    */
    public function findAll(): array {
        $ordersCollection = DB::table('orders')->get();

        return $this->em->orders->fromCollection($ordersCollection);
    }
    
    public function findById(int $id): ?Order {
        return $this->em->orders->findBy('id', $id);
    }
}
```