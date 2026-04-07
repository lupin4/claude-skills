# Module Design Patterns Catalog

## Pattern 1: Type + Operations Module

```fortran
module fm_vector
  implicit none
  private
  public :: ft_vec3_t, fm_vec3_add, fm_vec3_dot, fm_vec3_cross, fm_vec3_norm
  type :: ft_vec3_t
    real(8) :: x, y, z
  end type
contains
  ! operations on ft_vec3_t
end module
```

## Pattern 2: Constants Module

```fortran
module fm_constants
  implicit none
  real(8), parameter :: PI = 3.141592653589793d0
  real(8), parameter :: TWO_PI = 6.283185307179586d0
  real(8), parameter :: DEG_TO_RAD = PI / 180.0d0
  real(8), parameter :: EPSILON = 1.0d-15
end module
```

## Pattern 3: Abstract Interface Module

```fortran
module fm_function_interfaces
  implicit none
  abstract interface
    function scalar_function(x) result(y)
      real(8), intent(in) :: x
      real(8) :: y
    end function
    function vector_function(x, n) result(y)
      integer, intent(in) :: n
      real(8), intent(in) :: x(n)
      real(8) :: y(n)
    end function
  end interface
end module
```

## Pattern 4: Facade Module (re-exports)

```fortran
module fm_physics
  use fm_rigid_body, only: fm_rigid_body_step, fm_rigid_body_init, ft_rigid_body_t
  use fm_collision, only: fm_broadphase, fm_narrowphase
  use fm_constraints, only: fm_solve_constraints
  implicit none
  private
  ! Re-export public API
  public :: fm_rigid_body_step, fm_rigid_body_init, ft_rigid_body_t
  public :: fm_broadphase, fm_narrowphase
  public :: fm_solve_constraints
end module
```

## Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Module | fm_{domain} | fm_rigid_body |
| Public function | fm_{domain}_{operation} | fm_rigid_body_step |
| Public type | ft_{domain}_{name}_t | ft_rigid_body_t |
| Private helper | _{operation} | _normalize_quat |
| Constant | UPPER_SNAKE | MAX_ITERATIONS |
| File | {module_name}.f90 | fm_rigid_body.f90 |
