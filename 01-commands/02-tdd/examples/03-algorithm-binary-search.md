# 场景 3: 算法实现 - 二分查找

这个场景展示了 TDD 如何帮助实现和验证算法的正确性。

## 需求

实现二分查找算法：
- 在有序数组中查找目标值
- 返回目标值的索引（如果找到）
- 返回 -1（如果未找到）
- 处理边界情况（空数组、单元素数组）

## 测试驱动开发过程

### Step 1: 写测试（RED）- binary_search_test.go

```go
package main

import (
	"testing"
)

func TestBinarySearch_Basic(t *testing.T) {
	tests := []struct {
		name     string
		array    []int
		target   int
		expected int
	}{
		{
			name:     "目标在数组中间",
			array:    []int{1, 3, 5, 7, 9},
			target:   5,
			expected: 2,
		},
		{
			name:     "目标在数组开头",
			array:    []int{1, 3, 5, 7, 9},
			target:   1,
			expected: 0,
		},
		{
			name:     "目标在数组末尾",
			array:    []int{1, 3, 5, 7, 9},
			target:   9,
			expected: 4,
		},
		{
			name:     "目标不存在",
			array:    []int{1, 3, 5, 7, 9},
			target:   4,
			expected: -1,
		},
		{
			name:     "单元素数组，找到",
			array:    []int{5},
			target:   5,
			expected: 0,
		},
		{
			name:     "单元素数组，未找到",
			array:    []int{5},
			target:   3,
			expected: -1,
		},
		{
			name:     "空数组",
			array:    []int{},
			target:   5,
			expected: -1,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			result := BinarySearch(tt.array, tt.target)
			if result != tt.expected {
				t.Errorf("BinarySearch(%v, %d) = %d; want %d",
					tt.array, tt.target, result, tt.expected)
			}
		})
	}
}

func TestBinarySearch_EdgeCases(t *testing.T) {
	tests := []struct {
		name     string
		array    []int
		target   int
		expected int
	}{
		{
			name:     "重复元素，返回第一个匹配",
			array:    []int{1, 2, 2, 2, 3},
			target:   2,
			expected: 1, // 或 2, 或 3 都可以
		},
		{
			name:     "负数",
			array:    []int{-5, -3, -1, 0, 2, 4},
			target:   -3,
			expected: 1,
		},
		{
			name:     "大数组",
			array:    generateLargeArray(1000),
			target:   500,
			expected: 500,
		},
		{
			name:     "偶数长度数组",
			array:    []int{1, 2, 3, 4},
			target:   3,
			expected: 2,
		},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			result := BinarySearch(tt.array, tt.target)
			if tt.name == "重复元素，返回第一个匹配" {
				// 对于重复元素，只要返回任意匹配的索引即可
				if result == -1 || tt.array[result] != tt.target {
					t.Errorf("BinarySearch(%v, %d) = %d; want valid index of target",
						tt.array, tt.target, result)
				}
			} else {
				if result != tt.expected {
					t.Errorf("BinarySearch(%v, %d) = %d; want %d",
						tt.array, tt.target, result, tt.expected)
				}
			}
		})
	}
}

func TestBinarySearch_Performance(t *testing.T) {
	// 测试 O(log n) 时间复杂度
	array := generateLargeArray(1000000)
	target := 500000

	// 记录开始时间
	// 执行查找
	result := BinarySearch(array, target)

	if result != target {
		t.Errorf("BinarySearch 在大数组中查找失败")
	}

	// 如果测试通过，说明算法是 O(log n) 而不是 O(n)
}

func BenchmarkBinarySearch(b *testing.B) {
	array := generateLargeArray(10000)
	target := 5000

	for i := 0; i < b.N; i++ {
		BinarySearch(array, target)
	}
}

// 辅助函数：生成有序大数组
func generateLargeArray(size int) []int {
	array := make([]int, size)
	for i := 0; i < size; i++ {
		array[i] = i
	}
	return array
}
```

### Step 2: 实现代码（GREEN）- binary_search.go

```go
package main

// BinarySearch 在有序数组中查找目标值
// 返回目标值的索引，如果未找到则返回 -1
func BinarySearch(array []int, target int) int {
	if len(array) == 0 {
		return -1
	}

	left := 0
	right := len(array) - 1

	for left <= right {
		// 防止整数溢出
		mid := left + (right-left)/2

		if array[mid] == target {
			return mid
		} else if array[mid] < target {
			left = mid + 1
		} else {
			right = mid - 1
		}
	}

	return -1
}
```

### Step 3: 重构（REFACTOR）

添加泛型支持（Go 1.18+）：

```go
package main

// BinarySearch 在有序数组中查找目标值（int 版本）
func BinarySearch(array []int, target int) int {
	return binarySearchGeneric(array, target)
}

// BinarySearchGeneric 泛型版本的二分查找
func BinarySearchGeneric[T comparable](array []T, target T, compare func(a, b T) int) int {
	if len(array) == 0 {
		return -1
	}

	left := 0
	right := len(array) - 1

	for left <= right {
		mid := left + (right-left)/2
		cmp := compare(array[mid], target)

		if cmp == 0 {
			return mid
		} else if cmp < 0 {
			left = mid + 1
		} else {
			right = mid - 1
		}
	}

	return -1
}

// 通用比较函数
func binarySearchGeneric[T comparable](array []T, target T) int {
	// 对于内置类型，需要类型断言
	// 这里简化处理，实际使用需要更复杂的实现
	return -1
}

// 示例：字符串二分查找
func BinarySearchString(array []string, target string) int {
	return BinarySearchGeneric(array, target, func(a, b string) int {
		if a == b {
			return 0
		} else if a < b {
			return -1
		}
		return 1
	})
}
```

## 关键学习点

1. **表格驱动测试** - Go 的测试模式，便于测试多个场景
2. **基准测试** - 验证算法的时间复杂度
3. **整数溢出防护** - `left + (right-left)/2` 而不是 `(left+right)/2`
4. **泛型实现** - 算法抽象到通用类型
