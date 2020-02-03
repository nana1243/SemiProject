## 1.mybatis

1. product.xml

   ```xml
   
   	<!-- Random 6 products for Main Page -->
   	<select id="select_main_new" resultType="product">
   		SELECT * FROM (SELECT * FROM PRODUCT WHERE p_new='y') 
   		WHERE ROWNUM <![CDATA[ < ]]> 7 ORDER BY DBMS_RANDOM.RANDOM()
   	</select>
   
   	<select id="select_main_best" resultType="product">
   		SELECT * FROM (SELECT * FROM PRODUCT WHERE p_best='y') 
   		WHERE ROWNUM <![CDATA[ < ]]> 7 ORDER BY DBMS_RANDOM.RANDOM()
   	</select>
   ```

2. search.xml

   ```xml
   	
   	<select id="count_search" parameterType="String" resultType="int">
   		SELECT COUNT(*) FROM
   		PRODUCT WHERE P_CATEGORY LIKE '%'||#{search}||'%'
   		OR P_NAME LIKE
   		'%'||#{search}||'%'
   	</select>
   ```

   

3. board.xml

   ```xml
   
   	<select id="mypageselect" parameterType="String" resultType="board">
   		SELECT * FROM BOARD WHERE u_id=#{u_id}
   	</select>
   ```

   

4.stat.xml

```xml
<select id="selectbestyearly" resultType="stat">
		
    SELECT EXTRACT(year FROM orders.o_regdate) "condition", SUM(cart.c_sum) "totSale" 
    FROM cart, product, orders 
    WHERE cart.p_id=product.p_id AND orders.o_id=cart.o_id
	AND p_best='y' AND cart.o_id != 'null'
	GROUP BY EXTRACT(year FROM orders.o_regdate) 
    ORDER BY EXTRACT(year FROM orders.o_regdate)
	</select>
```





## 2.Controller

1. OrderDetail

   ```java
   @RequestMapping("/odetail.sp")
   	public ModelAndView odetail(HttpServletRequest request) {
   		ModelAndView mv = new ModelAndView();
   		
   		HttpSession session = request.getSession();
   	      String u_id = (String) session.getAttribute("loginid");  
   
   		try {			
   			
   			ArrayList<OrderVO> orderlist = obiz.getmyorder(u_id);
   			ArrayList<CartVO> carts = new ArrayList<CartVO>();
   						
   			for(OrderVO o: orderlist) {
   				String o_id = o.getO_id();
   				try {
   					carts.addAll(cbiz.getbyorder(o_id));
   				} catch (Exception e) {
   					e.printStackTrace();
   				}
   			}
   					   
   		   ArrayList<ProductVO> prod_o_detail_list = new ArrayList<ProductVO>();
   
   			for(CartVO c: carts) {
   				String p_id=c.getP_id();				
   				try {
   					prod_o_detail_list.add(pbiz.get(p_id));
   				} catch (Exception e) {
   					e.printStackTrace();
   				}
   			}
   			
   			mv.addObject("cartlist", carts);
   			mv.addObject("prod_o_detail_list", prod_o_detail_list);
   			mv.addObject("center", "user/orderdetail");
   
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   		mv.setViewName("main");
   		return mv;
   	}
   	
   ```



2. Board

   ```java
   // User 개인 Q&A
   	@RequestMapping("/mypage_qna.sp")
   	public ModelAndView mypage_qna(ModelAndView mv, HttpServletRequest request) {
   		
   		HttpSession session = request.getSession();
   		
   		String u_id = (String) session.getAttribute("loginid");	
   		System.out.print(u_id);
   		ArrayList<BoardVO> list = new ArrayList<BoardVO>();
   
   		try {
   			list = bbiz.mypageget(u_id);
   			System.out.println(list.toString());
   			mv.addObject("my_blist", list);
   			mv.addObject("center", "user/mypage_qna");
   		} catch (Exception e) {
   			e.printStackTrace();
   		}
   		mv.setViewName("main");
   		return mv;
   	}
   ```

   

3.main

```java
@RequestMapping("/main.sp")
	public ModelAndView main(HttpServletRequest request) {
		ModelAndView mv = new ModelAndView();
		try {
			ArrayList<ProductVO> bestlist = pbiz.get_main_new();
			ArrayList<ProductVO> newlist = pbiz.get_main_best();
			mv.addObject("jsp_bestlist", bestlist);
			mv.addObject("jsp_newlist", newlist);
		} catch (Exception e) {
			e.printStackTrace();
		}
		mv.setViewName("main");
		return mv;
	}
```

