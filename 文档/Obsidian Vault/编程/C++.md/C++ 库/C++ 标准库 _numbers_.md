# C++ 标准库 <numbers>

​`std::numbers`​ 是 C++20 中引入的一个标准库模块，主要用于提供一组常用的数学常量。

​`std::numbers`​位于 `<numbers>`​ 头文件中，并且包含了很多数学常量，涵盖了圆周率、自然对数的底数、黄金比例等常见常数。

C++ 使用这些常量可以提高代码的可读性、精度和效率，避免了重复定义和手动输入常数值。

### std::numbers 模块包含的常量

std::numbers 命名空间下包含以下成员：

```
template<floating_point T> inline constexpr T e_v<T> // 自然常数e
template<floating_point T> inline constexpr T log2e_v<T> // log2e
template<floating_point T> inline constexpr T log10e_v<T> // log10e
template<floating_point T> inline constexpr T pi_v<T> // 圆周率π
template<floating_point T> inline constexpr T inv_pi_v<T> // 1/π
template<floating_point T> inline constexpr T inv_sqrtpi_v<T> //1/根号π
template<floating_point T> inline constexpr T ln2_v<T> // ln2
template<floating_point T> inline constexpr T ln10_v<T> // ln10
template<floating_point T> inline constexpr T sqrt2_v<T> //根号2
template<floating_point T> inline constexpr T sqrt3_v<T> //根号3
template<floating_point T> inline constexpr T inv_sqrt3_v<T> // 1/根号3
template<floating_point T> inline constexpr T egamma_v<T> // 欧拉常数γ
template<floating_point T> inline constexpr T phi_v<T> // 黄金分割比Φ
inline constexpr double e = e_v<double>
inline constexpr double log2e = log2e_v<double>
inline constexpr double log10e = log10e_v<double>
inline constexpr double pi = pi_v<double>
inline constexpr double inv_pi = pi_v<double>
inline constexpr double inv_sqrtpi = pi_v<double>
inline constexpr double ln2 = ln2_v<double>
inline constexpr double ln10 = ln10_v<double>
inline constexpr double sqrt2 = sqrt2_v<double>
inline constexpr double sqrt3 = sqrt3_v<double>
inline constexpr double inv_sqrt3 = sqrt3_v<double>
inline constexpr double egamma = egamma_v<double>
inline constexpr double phi = phi_v<double>
```

以下是 std::numbers 中包含的常量列表：

|常量/模板名称|描述|示例值（近似）|
| ---------------| ---------------------------------------------------| ----------------|
|​`e_v`​|数学常数 e（自然对数的底数）|​`2.718281828459045`​|
|​`log2e_v`​|以 2 为底的 e 的对数（log₂(e)）|​`1.4426950408889634`​|
|​`log10e_v`​|以 10 为底的 e 的对数（log₁₀(e)）|​`0.4342944819032518`​|
|​`pi_v`​|数学常数 π（圆周率）|​`3.141592653589793`​|
|​`inv_pi_v`​|1/π（π的倒数）|​`0.318309886183121`​|
|​`inv_sqrtpi_v`​|1/√π（π的平方根的倒数）|​`0.5641895835477563`​|
|​`ln2_v`​|自然对数底数 2 的对数（ln(2)）|​`0.6931471805599453`​|
|​`ln10_v`​|自然对数底数 10 的对数（ln(10)）|​`2.302585092994046`​|
|​`sqrt2_v`​|√2（根号 2）|​`1.4142135623730951`​|
|​`sqrt3_v`​|√3（根号 3）|​`1.7320508075688772`​|
|​`inv_sqrt3_v`​|1/√3（根号 3 的倒数）|​`0.5773502691896257`​|
|​`egamma_v`​|欧拉-马歇罗尼常数 γ（Euler-Mascheroni constant）|​`0.5772156649015329`​|
|​`phi_v`​|黄金比例 Φ（(1 + √5) / 2）|​`1.618033988749895`​|
|​`e`​|常量 e (等价于 `e_v<double>`​)|​`2.718281828459045`​|
|​`log2e`​|常量 log₂(e) (等价于 `log2e_v<double>`​)|​`1.4426950408889634`​|
|​`log10e`​|常量 log₁₀(e) (等价于 `log10e_v<double>`​)|​`0.4342944819032518`​|
|​`pi`​|常量 π (等价于 `pi_v<double>`​)|​`3.141592653589793`​|
|​`inv_pi`​|常量 1/π (等价于 `inv_pi_v<double>`​)|​`0.318309886183121`​|
|​`inv_sqrtpi`​|常量 1/√π (等价于 `inv_sqrtpi_v<double>`​)|​`0.5641895835477563`​|
|​`ln2`​|常量 ln(2) (等价于 `ln2_v<double>`​)|​`0.6931471805599453`​|
|​`ln10`​|常量 ln(10) (等价于 `ln10_v<double>`​)|​`2.302585092994046`​|
|​`sqrt2`​|常量 √2 (等价于 `sqrt2_v<double>`​)|​`1.4142135623730951`​|
|​`sqrt3`​|常量 √3 (等价于 `sqrt3_v<double>`​)|​`1.7320508075688772`​|
|​`inv_sqrt3`​|常量 1/√3 (等价于 `inv_sqrt3_v<double>`​)|​`0.5773502691896257`​|
|​`egamma`​|常量 γ (欧拉-马歇罗尼常数) (等价于 `egamma_v<double>`​)|​`0.5772156649015329`​|
|​`phi`​|常量黄金比例 Φ (等价于 `phi_v<double>`​)|​`1.618033988749895`​|

显示详细信息

这些常量和变量模板涵盖了圆周率、自然对数底、黄金比例等常用的数学常数，并通过不同类型的变量模板提供精度选项，如 float、double 和 long double。
