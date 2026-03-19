# 场景 5: 领域模型 - 购物车

这个场景展示了 TDD 如何帮助实现复杂的领域模型和业务规则。

## 需求

实现电商购物车系统：
- 添加商品到购物车
- 移除商品
- 更新商品数量
- 计算总价（含税）
- 应用优惠券
- 检查库存
- 计算运费

## 测试驱动开发过程

### Step 1: 写测试（RED）- ShoppingCartTest.java

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.BeforeEach;
import static org.junit.jupiter.api.Assertions.*;

import java.math.BigDecimal;
import java.util.Currency;

class ShoppingCartTest {

    private Product product1;
    private Product product2;
    private Coupon coupon10Percent;

    @BeforeEach
    void setUp() {
        product1 = new Product("P001", "商品1", new BigDecimal("100.00"), 10);
        product2 = new Product("P002", "商品2", new BigDecimal("50.00"), 5);
        coupon10Percent = new Coupon("SAVE10", new BigDecimal("0.10"), CouponType.PERCENTAGE);
    }

    @Test
    void shouldAddProductToCart() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);

        assertEquals(1, cart.getItemCount());
        assertEquals(2, cart.getQuantity(product1));
    }

    @Test
    void shouldIncreaseQuantityWhenProductAlreadyExists() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);
        cart.addProduct(product1, 3);

        assertEquals(1, cart.getItemCount());
        assertEquals(5, cart.getQuantity(product1));
    }

    @Test
    void shouldRemoveProductFromCart() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);
        cart.removeProduct(product1);

        assertEquals(0, cart.getItemCount());
    }

    @Test
    void shouldCalculateSubtotal() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2); // 100 * 2 = 200
        cart.addProduct(product2, 3); // 50 * 3 = 150

        assertEquals(new BigDecimal("350.00"), cart.getSubtotal());
    }

    @Test
    void shouldCalculateTax() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);
        cart.setTaxRate(new BigDecimal("0.08")); // 8% 税率

        // 200 * 0.08 = 16.00
        assertEquals(new BigDecimal("16.00"), cart.getTax());
    }

    @Test
    void shouldCalculateTotalIncludingTax() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2); // 200
        cart.setTaxRate(new BigDecimal("0.08")); // 8% 税率

        // 200 + 16 = 216
        assertEquals(new BigDecimal("216.00"), cart.getTotal());
    }

    @Test
    void shouldApplyPercentageDiscountCoupon() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2); // 200
        cart.setTaxRate(new BigDecimal("0.08"));
        cart.applyCoupon(coupon10Percent); // 10% 折扣

        // 折扣: 200 * 0.10 = 20
        // 小计: 200 - 20 = 180
        // 税: 180 * 0.08 = 14.40
        // 总计: 180 + 14.40 = 194.40
        assertEquals(new BigDecimal("194.40"), cart.getTotal());
    }

    @Test
    void shouldApplyFixedAmountCoupon() {
        Coupon coupon20Off = new Coupon("SAVE20", new BigDecimal("20.00"), CouponType.FIXED_AMOUNT);
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2); // 200
        cart.applyCoupon(coupon20Off);

        assertEquals(new BigDecimal("180.00"), cart.getSubtotal());
    }

    @Test
    void shouldNotAllowNegativeQuantity() {
        ShoppingCart cart = new ShoppingCart();

        assertThrows(IllegalArgumentException.class, () -> {
            cart.addProduct(product1, -1);
        });
    }

    @Test
    void shouldNotAllowQuantityExceedingStock() {
        ShoppingCart cart = new ShoppingCart();
        // product1 库存只有 10

        assertThrows(InsufficientStockException.class, () -> {
            cart.addProduct(product1, 11);
        });
    }

    @Test
    void shouldCalculateShippingCost() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);
        cart.setShippingCalculator(new FlatRateShippingCalculator(new BigDecimal("10.00")));

        assertEquals(new BigDecimal("10.00"), cart.getShippingCost());
    }

    @Test
    void shouldOfferFreeShippingOverMinimumAmount() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 5); // 500
        cart.setShippingCalculator(new FreeShippingThresholdCalculator(
            new BigDecimal("15.00"),
            new BigDecimal("400.00")
        ));

        // 订单超过 400，免运费
        assertEquals(BigDecimal.ZERO, cart.getShippingCost());
    }

    @Test
    void shouldCombineAllCosts() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2); // 200
        cart.setTaxRate(new BigDecimal("0.08"));
        cart.applyCoupon(coupon10Percent);
        cart.setShippingCalculator(new FlatRateShippingCalculator(new BigDecimal("10.00")));

        // 小计: 180 (折扣后)
        // 税: 14.40
        // 运费: 10.00
        // 总计: 204.40
        assertEquals(new BigDecimal("204.40"), cart.getTotal());
    }

    @Test
    void shouldClearCart() {
        ShoppingCart cart = new ShoppingCart();
        cart.addProduct(product1, 2);
        cart.addProduct(product2, 1);
        cart.clear();

        assertEquals(0, cart.getItemCount());
        assertEquals(BigDecimal.ZERO, cart.getSubtotal());
    }
}
```

### Step 2: 实现代码（GREEN）- ShoppingCart.java

```java
import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.HashMap;
import java.util.Map;

public class ShoppingCart {
    private final Map<Product, Integer> items = new HashMap<>();
    private BigDecimal taxRate = BigDecimal.ZERO;
    private Coupon appliedCoupon;
    private ShippingCalculator shippingCalculator;

    public void addProduct(Product product, int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("数量必须大于零");
        }
        if (quantity > product.getStock()) {
            throw new InsufficientStockException("库存不足");
        }

        items.merge(product, quantity, Integer::sum);
    }

    public void removeProduct(Product product) {
        items.remove(product);
    }

    public void updateQuantity(Product product, int quantity) {
        if (quantity <= 0) {
            removeProduct(product);
            return;
        }
        if (quantity > product.getStock()) {
            throw new InsufficientStockException("库存不足");
        }

        items.put(product, quantity);
    }

    public int getItemCount() {
        return items.size();
    }

    public int getQuantity(Product product) {
        return items.getOrDefault(product, 0);
    }

    public BigDecimal getSubtotal() {
        BigDecimal subtotal = BigDecimal.ZERO;
        for (Map.Entry<Product, Integer> entry : items.entrySet()) {
            Product product = entry.getKey();
            int quantity = entry.getValue();
            subtotal = subtotal.add(product.getPrice().multiply(BigDecimal.valueOf(quantity)));
        }

        if (appliedCoupon != null) {
            subtotal = applyCoupon(subtotal);
        }

        return subtotal.setScale(2, RoundingMode.HALF_UP);
    }

    private BigDecimal applyCoupon(BigDecimal amount) {
        if (appliedCoupon.getType() == CouponType.PERCENTAGE) {
            return amount.multiply(BigDecimal.ONE.subtract(appliedCoupon.getValue()));
        } else {
            // 固定金额折扣
            BigDecimal discounted = amount.subtract(appliedCoupon.getValue());
            return discounted.compareTo(BigDecimal.ZERO) > 0 ? discounted : BigDecimal.ZERO;
        }
    }

    public BigDecimal getTax() {
        return getSubtotal().multiply(taxRate).setScale(2, RoundingMode.HALF_UP);
    }

    public BigDecimal getShippingCost() {
        if (shippingCalculator == null) {
            return BigDecimal.ZERO;
        }
        return shippingCalculator.calculate(getSubtotal()).setScale(2, RoundingMode.HALF_UP);
    }

    public BigDecimal getTotal() {
        return getSubtotal()
                .add(getTax())
                .add(getShippingCost())
                .setScale(2, RoundingMode.HALF_UP);
    }

    public void setTaxRate(BigDecimal taxRate) {
        this.taxRate = taxRate;
    }

    public void applyCoupon(Coupon coupon) {
        this.appliedCoupon = coupon;
    }

    public void setShippingCalculator(ShippingCalculator shippingCalculator) {
        this.shippingCalculator = shippingCalculator;
    }

    public void clear() {
        items.clear();
        taxRate = BigDecimal.ZERO;
        appliedCoupon = null;
        shippingCalculator = null;
    }
}

// 支持类
class Product {
    private String id;
    private String name;
    private BigDecimal price;
    private int stock;

    public Product(String id, String name, BigDecimal price, int stock) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public BigDecimal getPrice() { return price; }
    public int getStock() { return stock; }
}

class Coupon {
    private String code;
    private BigDecimal value;
    private CouponType type;

    public Coupon(String code, BigDecimal value, CouponType type) {
        this.code = code;
        this.value = value;
        this.type = type;
    }

    public String getCode() { return code; }
    public BigDecimal getValue() { return value; }
    public CouponType getType() { return type; }
}

enum CouponType {
    PERCENTAGE,
    FIXED_AMOUNT
}

interface ShippingCalculator {
    BigDecimal calculate(BigDecimal orderTotal);
}

class FlatRateShippingCalculator implements ShippingCalculator {
    private BigDecimal rate;

    public FlatRateShippingCalculator(BigDecimal rate) {
        this.rate = rate;
    }

    @Override
    public BigDecimal calculate(BigDecimal orderTotal) {
        return rate;
    }
}

class FreeShippingThresholdCalculator implements ShippingCalculator {
    private BigDecimal defaultRate;
    private BigDecimal freeThreshold;

    public FreeShippingThresholdCalculator(BigDecimal defaultRate, BigDecimal freeThreshold) {
        this.defaultRate = defaultRate;
        this.freeThreshold = freeThreshold;
    }

    @Override
    public BigDecimal calculate(BigDecimal orderTotal) {
        if (orderTotal.compareTo(freeThreshold) >= 0) {
            return BigDecimal.ZERO;
        }
        return defaultRate;
    }
}

class InsufficientStockException extends RuntimeException {
    public InsufficientStockException(String message) {
        super(message);
    }
}
```

### Step 3: 重构（REFACTOR）

使用策略模式重构计算逻辑：

```java
public class ShoppingCart {
    private final Map<Product, Integer> items = new HashMap<>();
    private TaxStrategy taxStrategy = new NoTaxStrategy();
    private DiscountStrategy discountStrategy = new NoDiscountStrategy();
    private ShippingStrategy shippingStrategy = new NoShippingStrategy();

    public void setTaxStrategy(TaxStrategy taxStrategy) {
        this.taxStrategy = taxStrategy;
    }

    public void setDiscountStrategy(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public void setShippingStrategy(ShippingStrategy shippingStrategy) {
        this.shippingStrategy = shippingStrategy;
    }

    public BigDecimal getTotal() {
        BigDecimal subtotal = calculateSubtotal();
        BigDecimal tax = taxStrategy.calculate(subtotal);
        BigDecimal shipping = shippingStrategy.calculate(subtotal);

        return subtotal.add(tax).add(shipping).setScale(2, RoundingMode.HALF_UP);
    }

    private BigDecimal calculateSubtotal() {
        BigDecimal total = BigDecimal.ZERO;
        for (Map.Entry<Product, Integer> entry : items.entrySet()) {
            total = total.add(entry.getKey().getPrice()
                    .multiply(BigDecimal.valueOf(entry.getValue())));
        }
        return discountStrategy.apply(total).setScale(2, RoundingMode.HALF_UP);
    }
}

// 策略接口
interface TaxStrategy {
    BigDecimal calculate(BigDecimal amount);
}

interface DiscountStrategy {
    BigDecimal apply(BigDecimal amount);
}

interface ShippingStrategy {
    BigDecimal calculate(BigDecimal orderTotal);
}

// 具体策略实现
class PercentageTaxStrategy implements TaxStrategy {
    private final BigDecimal rate;

    public PercentageTaxStrategy(BigDecimal rate) {
        this.rate = rate;
    }

    @Override
    public BigDecimal calculate(BigDecimal amount) {
        return amount.multiply(rate).setScale(2, RoundingMode.HALF_UP);
    }
}

class PercentageDiscountStrategy implements DiscountStrategy {
    private final BigDecimal percentage;

    public PercentageDiscountStrategy(BigDecimal percentage) {
        this.percentage = percentage;
    }

    @Override
    public BigDecimal apply(BigDecimal amount) {
        return amount.multiply(BigDecimal.ONE.subtract(percentage));
    }
}
```

## 关键学习点

1. **领域模型** - ShoppingCart、Product、Coupon 等都是领域概念
2. **策略模式** - 税收、折扣、运费计算使用策略模式
3. **BigDecimal 精度计算** - 金融计算必须使用 BigDecimal
4. **不变式验证** - 数量不能为负，不能超过库存
