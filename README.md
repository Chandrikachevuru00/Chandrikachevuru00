Create a Spring Boot project and define the Product entity:
java


@Entity
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long productId;

    private String name;
    private String description;
    private BigDecimal price;
    private int quantityAvailable;

    // getters and setters
}
Create a ProductRepository interface for basic CRUD operations:

public interface ProductRepository extends JpaRepository<Product, Long> {
    // Custom queries if needed
}


Implement a ProductService to handle business logic:
@Service
public class ProductService {

    @Autowired
    private ProductRepository productRepository;

    public Product createProduct(Product product) {
        // Implement validation and error handling as needed
        return productRepository.save(product);
    }

    public Product readProduct(Long productId) {
        return productRepository.findById(productId).orElse(null);
    }

    public String updateProduct(Product product) {
        // Implement validation and error handling as needed
        if (productRepository.existsById(product.getProductId())) {
            productRepository.save(product);
            return "Product updated successfully";
        } else {
            return "Product not found";
        }
    }

    public String deleteProduct(Long productId) {
        // Implement validation and error handling as needed
        if (productRepository.existsById(productId)) {
            productRepository.deleteById(productId);
            return "Product deleted successfully";
        } else {
            return "Product not found";
        }
    }

    public String applyDiscountOrTax(Long productId, BigDecimal rate, boolean isDiscount) {
        // Implement validation and error handling as needed
        Product product = productRepository.findById(productId).orElse(null);

        if (product != null) {
            BigDecimal modifiedPrice = isDiscount
                    ? product.getPrice().multiply(BigDecimal.ONE.subtract(rate.divide(BigDecimal.valueOf(100))))
                    : product.getPrice().multiply(BigDecimal.ONE.add(rate.divide(BigDecimal.valueOf(100))));

            product.setPrice(modifiedPrice);
            productRepository.save(product);
            return "Discount/Tax applied successfully. Updated product details: " + product.toString();
        } else {
            return "Product not found";
        }
    }
}

Create a ProductController for handling API endpoin
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @Autowired
    private ProductService productService;

    @PostMapping
    public ResponseEntity<Object> createProduct(@RequestBody Product product) {
        Product createdProduct = productService.createProduct(product);
        return new ResponseEntity<>(createdProduct, HttpStatus.CREATED);
    }

    @GetMapping("/{productId}")
    public ResponseEntity<Object> readProduct(@PathVariable Long productId) {
        Product product = productService.readProduct(productId);
        if (product != null) {
            return new ResponseEntity<>(product, HttpStatus.OK);
        } else {
            return new ResponseEntity<>("Product not found", HttpStatus.NOT_FOUND);
        }
    }

    @PutMapping
    public ResponseEntity<Object> updateProduct(@RequestBody Product product) {
        String result = productService.updateProduct(product);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }

    @DeleteMapping("/{productId}")
    public ResponseEntity<Object> deleteProduct(@PathVariable Long productId) {
        String result = productService.deleteProduct(productId);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }

    @PatchMapping("/{productId}/apply-discount")
    public ResponseEntity<Object> applyDiscount(@PathVariable Long productId,
                                               @RequestParam BigDecimal discountRate) {
        String result = productService.applyDiscountOrTax(productId, discountRate, true);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }

    @PatchMapping("/{productId}/apply-tax")
    public ResponseEntity<Object> applyTax(@PathVariable Long productId,
                                          @RequestParam BigDecimal taxRate) {
        String result = productService.applyDiscountOrTax(productId, taxRate, false);
        return new ResponseEntity<>(result, HttpStatus.OK);
    }
}
Create a ProductController for handling API endpoin
