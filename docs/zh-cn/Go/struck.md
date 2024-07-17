```
package main

import "fmt"

type Task struct {
	//Handlers []func(*Context)
	job []func(string) string
	//	job  func (x,y int) int {
	//	return x + y
	//}
	name  string
	index int8
}

func (p *Task) Next() {
	p.index++
	p.job[p.index]("test")
}

// func add(x, y int) int { // 定义加法函数
//
//		return x + y
//	}
func main() {
	a := &Task{}
	a.job = make([]func(string) string, 0)
	//
	//// 注册中间件
	a.job = append(a.job, mq)
	//c.job = append(c.job, m2)
	//c.job = append(c.job, m3)
	//
	//// 控制器函数
	//c.job = append(c.job, action)

	// 启动
	a.job[0]("test")
}

//	func action(c *Task) {
//		fmt.Println("main handler")
//	}
//
//	func m1(c *Task) {
//		fmt.Println("m1 start")
//		//c.Next()
//	}
func mq(e string) string {
	//fmt.Println("stringtest")
	fmt.Printf("this is a test: %v\n", e)
	return e
	e.Next()
}

//
//func m2(c *Task) {
//	fmt.Println("m2 start")
//	c.Next()
//	fmt.Println("m2 end")
//}
//
//func m3(c *Task) {
//	c.Next()
//	fmt.Println("m3 end")
//}

```
