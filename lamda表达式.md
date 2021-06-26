// mutable Lambda                                                              //
// in default satuation,Lamba don't change the value that is captured by value //
// add keywords befor paramater list can change this situation                 //
void func()
{
    size_t v1 = 41;
    auto f = [v1] () {return ++v1;};
    v1 = 0;
    auto j = f();                                                              //j == 42;
}
